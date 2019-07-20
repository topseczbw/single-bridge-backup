# 使用 postMessage 解决 iframe 跨域通信问题

1. ## 调用方

   ### 注意

   使用url传参给iframe进行项目初始化，不要使用 `postMessage` 。

   ```js
   async start() {
         await this.login()
         const _this = this
         const iframeEle = document.createElement('iframe')
         const wrapEle = document.querySelector('.example__content')
         
         // 
         /* 等iframe加载完成后，向iframe发送数据。z
         iframeEle.addEventListener('load', function() {
           iframeEle.contentWindow.postMessage(_this.config, '*')
         })
         */
         const params = this.getParams()
         iframeEle.src =
           location.origin +
           (isDevEnv ? `/index.html?${params}#/` : `/diy/index.html?${params}#/`)
         iframeEle.style.cssText += 'width: 100%;height: 100%;border: 0;'
         wrapEle.innerHTML = ''
         wrapEle.appendChild(iframeEle)
       },
         
     // 监听返回结果
     async created() {
       window.addEventListener(
         'message',
         function(e) {
           if (e.data.source === 'diy') {
             console.log('paper收到来自diy的数据')
             document.querySelector('.result').innerHTML = `<p>生成试题ID:</p>${
               e.data.queId
             }`
           }
         },
         false
       )
     }
   ```

   

2. ## 服务方

   通过url获取参数进行项目初始化。

   使用 `window.parent` 获取到调用方窗口。

   ```js
   // 通过url获取参数进行项目初始化
   function parseQueryString(url) {
     url = url == null ? window.location.href : url
     var search = url.substring(url.lastIndexOf('?') + 1)
     if (!search) {
       return {}
     }
     return JSON.parse('{"' + decodeURIComponent(search).replace(/"/g, '\\"').replace(/&/g, '","').replace(/=/g, '":"') + '"}')
   }
   
   // 删除末尾 "#/"
   var global_initConfig = (function() {
     try {
       var url = window.location.href.slice(0, -2)
       return parseQueryString(url)
     } catch (e) {
     }
   })()
   
   // 向调用方传递数据
   postMessageEvent(queId) {
     window.parent.postMessage(
       {
         source: 'diy',
         queId
       },
       '*'
     )
   }
   ```

   