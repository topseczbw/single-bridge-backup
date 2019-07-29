# js基础

1. ## encodeURIComponent 与 decodeURIComponent

   ### 是什么？

   用于URI的编码和解码

   ### 为什么存在？

   为了避免服务器收到不可预知的请求，对任何用户输入的作为URI部分的内容你都需要用encodeURIComponent进行转义。比如，一个用户可能会输入"`Thyme &time=again`"作为`comment`变量的一部分。如果不使用encodeURIComponent对此内容进行转义，服务器得到的将是`comment=Thyme%20&time=again`。请注意，"&"符号和"="符号产生了一个新的键值对，所以服务器得到两个键值对（一个键值对是`comment=Thyme`，另一个则是`time=again`），而不是一个键值对。

   ### 在哪里使用？

   通过URL方式发送GET请求时，现将 `url `编码。

2. ## getBoundingClientRect

   **Element.getBoundingClientRect**方法返回元素的大小及其相对于视口的位置。

   返回值是一个 [DOMRect](https://developer.mozilla.org/zh-CN/docs/Mozilla/Tech/XPCOM/Reference/Interface/nsIDOMClientRect) 对象，DOMRect 对象包含了一组用于描述边框的只读属性—left、top、right和bottom，单位为像素。除了 width 和 height 外的属性都是**相对于视口的左上角**位置而言的。

3. ## dispatch 派发自定义事件

   ```js
   // 核心使用 dispatchEvent API
   const event = new Event('zbw')
   const body = document.querySelector('body')
   body.addEventListener('zbw', ev => {
   	console.log('收到全局自定义事件')
   }, false)
   body.addEventListener('click', ev => {
   	body.dispatchEvent(event)
   }, false)
   ```

4. ## mousedown、mouseup、click事件之间的关系及执行顺序

   **mousedown** 鼠标按下触发，**不区分左右键**。

   **mouseup** 鼠标松开触发，当鼠标指针位于元素上方时，放松鼠标按钮就会触发该事件，即使mousedown时不是该元素也可以触发。**不区分左右键**。

   **click** 按下并松开后触发，**只作用于鼠标左键**

   **执行顺序**是 mousedown — mouseup — click

5. ## 什么是字符实体？

   &times;  **x**

6. ## IIFE（立即调用函数表达式）是什么？

7. ## 重绘和重排

8. ## 字符编码