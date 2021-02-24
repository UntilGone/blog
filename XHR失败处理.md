# 前端如何友好的处理xhr失败
## 需求
1. 请求失败（包括http和业务逻辑上的失败），做出友好合理的提示
2. 对于登录态失效等特殊情况，避免多个请求同时失败，重复提示
3. 避免重复写提示部分代码
4. 有公共的异常处理，也可以在业务代码中处理异常的业务场景
## 方案
### 规范接口响应体
**最重要的一步**，需要与后端定义好response data的内容；
```typescript
// response
{
  data:{
    data:any; // 实际的数据，可以是数组、对象、字符串等等
    code:number;  // 请求状态码，用于判断接口是否成功（这里的成功是只业务层面的，不是http层面）
    message:string; // 消息，用于前端展示（主要是异常场景）
  },
  // 其他属性
}
```
### 统一提示
解决重复写提示信息部分代码很好解决，直接在请求的拦截器里处理就好。因为使用了axios，所以就拿axios来做例子。
```javascript
import axios from 'axios';
// axios实例
export const http = axios.create({
  baseURL: '/',
  timeout: 5000,
  withCredentials: true,
  validateStatus: (status) => status >= 200 && status < 400, // http状态码的拦截，根据接口规则随机应变
});
// response拦截器
http.interceptors.response.use((res) => {
  // 与后端商定，接口code不为200均为不正常
  if (res.data.code !== 200) {
    // 公共异常业务处理，如登录态失效
    // 登录态失效code码为401
    if (res.data.code === 401) {
      // 清除缓存、数据等
      initData();
      history.push('/login'); // 进入登录页或者首页之类的
      message.error('登录失效，请重新登录'); // 提示用户
      throw new Error('LOGIN_FAILED');  // 抛出错误码
    }
    // 提示异常信息
    message.error(res.data.message);
    // 抛出错误码，业务代码中可以根据catch到的code做业务处理
    throw new Error(res.data.code);
  }
  // 这边可以使用 return res.data; 因为大多数场景下业务中只需要使用res.data
  // 这样业务代码就只用res.data就可以获取数据，不需要res.data.data;
  return res;
}, (err) => {
  // 处理axios的拦截器，主要就是上面状态码的拦截
  // 定义一些常见错误码，比如Network Error或者timeout之类的
  if (err.message === 'Network Error') {
    message.error('网络异常，请检查后重试');
  } else {
    err.message && message.error(err.message);
  }
  throw new Error(err);
});
```

### ErrorMessage处理
上面代码中是直接弹出res.data.message，也就是后端给什么前端就提示什么。这样的就会带来另外的问题：多语言的处理；不同项目调用同一个服务提示的文案不同的场景等待。
为了解决这个问题，前端可以在每个项目里定义一个MessageMap，用来映射后端抛出的message。这样就可以通过i18n之类的库轻松做到多语言，也可以满足不同项目中对同一message提示不同内容。为了避免在多个项目中重复写MessageMap，可以考虑使用公共的远程文件/npm库来提供公共内容，在通过自定义内容覆盖合并满足自定义。

### 取消请求
进入页面时同时发起多个异步请求是很常见的场景。如果这个时候用户的登录态失效了，这样会发起的多个请求，实际上都会失败。会造成资源浪费（给服务器发出了不必要的请求），前端也会做出多个提示（当然这个可以通过其他方案解决）。这都是我们不希望看到的，解决这个问题有2个方案：第一就是提供一个同步接口，接口成功了才继续渲染；第二个就是当有一个请求失败了，就取消其他请求。这里我选择第二个方案，第一个方案容易引起首屏渲染或者等待交互时间过长。
直接上代码:
```javascript
// axios提供了一个cancel token用于取消某一个请求
// 定义一个cancel token
let source = axios.CancelToken.source();
// request拦截器
http.interceptors.request.use((config) => {
  // 为当前请求设置cancel Token，后面会讲为什么不直接在创建axios实例的时候设置
  config.cancelToken = source.token;
  return config;
}, (err) => err);
http.interceptors.response.use((res) => {
  if (res.data.code !== 200) {
    if (res.data.code === 401) {
      // 登录失效了 取消其他请求
      source.cancel();
      // 重置token
      source = axios.CancelToken.source();
      message.error('登录失效，请重新登录');
      throw new Error('登录失效，请重新登录');
    } else {
      message.error(res.data.message);
      throw new Error(res.data.message);
    }
  }
  return res;
}, (err) => {
  // handle error
});
```
axios创建实例的时候可以直接设置Cancel Token，没有这么做是因为当执行了source.cancel()后，所有是这个token的请求都会被取消，无论发起时间。也就是说，如果不改变cancel token，在执行source.cancel()后，所有请求都无法发出，直接被取消。通过在发起请求时设置一次当前的token，可以避免这个问题。
上面代码还有个问题，就是如果需要针对某一个请求进行取消的时候，因为在request拦截器里做了处理，所以需要进行一些处理。
```javascript
http.interceptors.request.use((config) => {
  // 没有才设置，有的话就用原本的
  if (!config.cancelToken) {
    config.cancelToken = source.token;
  }
  return config;
}, (err) => err);
```

## 总结
这样已经满足了开头所述的需求，这么做的前提是和后端规范好API规范，例如使用RESTful或者怎么样。否则，前端只能针对不同接口进行不同处理。最后贴个完整代码:
http.js
```javascript
// 申明axios实例
import axios from 'axios';
import message from 'utils/message';  // toast方法
import { errorMessage } from './error-message';
// axios实例
export const http = axios.create({
  baseURL: '/',
  timeout: 5000,
  withCredentials: true,
  validateStatus: (status) => status >= 200 && status < 400, // http状态码的拦截，根据接口规则随机应变
});
// 申明cancelToken
let source = axios.CancelToken.source();
// request拦截器
http.interceptors.request.use((config) => {
  // 为当前请求设置cancel Token
  config.cancelToken = source.token;
  return config;
}, (err) => err); // 这边的error也可以做处理
// response拦截器
http.interceptors.response.use((res) => {
  // 与后端商定，接口code不为200均为不正常
  if (res.data.code !== 200) {
    // 公共异常业务处理，如登录态失效
    // 登录态失效code码为401
    if (res.data.code === 401) {
      // 登录失效了 取消其他请求
      source.cancel();
      // 重置token
      source = axios.CancelToken.source();
      // 清除缓存、数据等
      initData();
      history.push('/login'); // 进入登录页或者首页之类的
      message.error('登录失效，请重新登录'); // 提示用户
      throw new Error('LOGIN_FAILED');  // 抛出错误码
    } else {
      // 提示异常信息
      message.error(errorMessage(res.data.message));
      // 抛出错误码，业务代码中可以根据catch到的code做业务处理
      throw new Error(res.data.code);
    }
    // 还可以根据需要进行打点
    log({
      msg: 'api error',
      path: 'query path',
      // 需要的数据
    });
  }
  // 这边可以使用 return res.data; 因为大多数场景下业务中只需要使用res.data
  // 这样业务代码就只用res.data就可以获取数据，不需要res.data.data;
  return res;
}, (err) => {
  // 处理axios的拦截器，主要就是上面状态码的拦截
  // 定义一些常见错误码，比如Network Error或者timeout之类的
  if (err.message === 'Network Error') {
    message.error('网络异常，请检查后重试');
  } else {
    err.message && message.error(err.message);
  }
  throw new Error(err);
});
```
error-message.js
```javascript
export const errorMessage = (msg:string) => {
  // key由后端定义 value由前端定义
  const messageMap = {
    'message1': '请求失败，请重试',
    'message2': '请求失败，请重试',
    'message3': '请求失败，请重试',
  };
  return messageMap[msg] || '未知错误，请重试';
}
```
业务代码
```javascript
const queryData = async () => {
  try {
    const res = await fetchData();
    // 业务逻辑
  } catch (err) {
    // 异常处理
    if (err.message === 'xxxx') {
      // 处理
    }
    // 业务打点
    log('xxx error', err);
  }
}
```