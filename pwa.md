# PWA
因为最近业务需要调研一下移动端的方案，就简单了解了一下PWA。  
## 简介
> PWA（Progressive web apps，渐进式 Web 应用）运用现代的 Web API 以及传统的渐进式增强策略来创建跨平台 Web 应用程序。这些应用无处不在、功能丰富，使其具有与原生应用相同的用户体验优势。 —— MDN
在我看来，PWA主要解决了这几个问题：
1. 使web app离线
2. 在用户设备上提供一个快捷、稳定的入口
3. 支持消息推送、应用通知

## 对比
相对于原生，PWA的优点在于迭代迅速，第一就是开发成本低，第二就是不需要通过App Store审核，而且有更低的更新成本。缺点也很明显，性能相对较差，与原生交互比较受限。  
相对于纯Web，PWA有稳定的入口，体验更像是原生。  

## 缺点
需要设备支持，虽然现在Android和IOS支持都不错，但是对于Android手机厂商的原生浏览器不一定支持，而且需要进行桌面授权，用户使用不方便。  
在demo测试过程中，IOS是可以直接通过safari添加到桌面。Android使用了华为和OPPO，原装浏览器都不支持直接添加（也有可能是我没找到），然后通过安装Chrome可以添加到桌面（需要授权，测试手机没有直接弹出授权，需要自己去设置里设置）。顺便一提，Android的体验比IOS的要好，backgroundColor之类的参数好像在IOS下无效（可能有别的参数代替）。

## PWA的特性支持
介绍一下实现PWA需要的的Web特性。
### App Manifest
Web应用程序清单在一个JSON文本文件中提供有关应用程序的信息（如名称，作者，图标和描述）。manifest 的目的是将Web应用程序安装到设备的主屏幕，为用户提供更快的访问和更丰富的体验。  
使用方式：
```html
<link rel="manifest" href="/manifest.json">
```
具体配置参考：https://developer.mozilla.org/zh-CN/docs/Web/Manifest  
兼容性支持：[Can I use](https://caniuse.com/?search=App%20Manifest)

### Service Worker
Service Worker 可以使你的应用先访问本地缓存资源，所以在离线状态时，在没有通过网络接收到更多的数据前，仍可以提供基本的功能（一般称为Offline First）。  
具体信息：https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers  
兼容性支持：[Can I use](https://caniuse.com/?search=Service%20Worker)

### Notifications API
Notifications API的通知接口用于向用户配置和显示桌面通知.  
详情：https://developer.mozilla.org/zh-CN/docs/Web/API/notification  
兼容性支持：[Can I use](https://caniuse.com/?search=Notifications)

### Push API
Push API 给与了Web应用程序接收从服务器发出的推送消息的能力，无论Web应用程序是否在用户设备前台，甚至刚加载完成。这样，开发人员就可以向用户投放异步通知和更新，从而让用户能更及时地获取新内容。  
了解详情：https://developer.mozilla.org/zh-CN/docs/Web/API/Push_API  
兼容性支持：[Can I use](https://caniuse.com/?search=Push)

### Background Sync
Background Sync允许延迟请求，直到用户连接稳定再执行。这对于确保用户想要发送的任何内容都是真正发送的非常有用。  
详情：https://developers.google.com/web/updates/2015/12/background-sync  
兼容性支持：[Can I use](https://caniuse.com/?search=Background%20Sync)

## 其他
webpack可以配合workbox-webpack-plugin实现，详情见：https://lavas-project.github.io/pwa-book