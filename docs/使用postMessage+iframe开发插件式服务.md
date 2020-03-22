# 使用postMessage+iframe开发插件式服务

## 概念

**调用方**：调用iframe服务的一方

**服务方**：在iframe中，开发页面服务的一方

## 调用方

1. 调用方使用url传参方式对iframe项目进行初始化

   ```js
   const iframeElement = document.createElement('iframe')
   const wrapElement = document.querySelector('.example__content');

   const params = this.getParams
   iframeElement.src = location.origin + params
   iframeElement.style.cssText += 'width: 700px; height: 500px; border: 0;'

   wrapElement.innerHTML = ''
   wrapElement.appendChild(iframe)
   ```

2. 监听服务方派发出的事件

   ```js
   window.addEventListener(
     'message',
     function(e) {
       if (e.data.source === '服务方唯一标识') {
         // 调用方收到来自服务方的数据
       }
     }
   )
   ```

3. 调用方通过postMessage向服务方传递数据

   ```js
   document.getElementById('Iframe').contentWindow.postMessage({
     source: '调用方唯一标识',
     eventType: '事件唯一标识'
   })
   ```

## 服务方

1. 服务方通过解析url获取参数，对项目进行初始化

   ```js
    // url参数转对象
    function parseQueryString(url) {
      url = url == null ? window.location.href : url
      var search = url.substring(url.lastIndexOf('?') + 1)
      if (!search) {
        return {}
      }
      return JSON.parse('{"' + decodeURIComponent(search).replace(/"/g, '\\"').replace(/&/g, '","').replace(/=/g, '":"') + '"}')
    }

   // 使用vue-router  hash模式时 需要删除末尾 "#/"
    var global_initConfig = (function() {
      try {
        var url = window.location.href.slice(0, -2)
        return parseQueryString(url)
      } catch (e) {
      }
    })()
   ```

1. 服务方派发消息给调用方

   ```js
   postMessageEvent(data) {
    window.parent.postMessage({
      source: '调用方唯一标识',
      data
    }, '*')
   }
   ```
