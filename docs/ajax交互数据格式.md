# ajax交互数据格式

2019/09/23 22:05

<!-- TOC -->

- [表单格式](#表单格式)
- [formData格式](#formdata格式)
- [json格式](#json格式)

<!-- /TOC -->

## 表单格式

`content-type:application/x-www-form-urlencoded`

格式为：`name=zbw&sex=1&email=569119225@qq.com` 字符串

示例：

```js
let xhr = new XMLHttpRequest()

xhr.onreadystatechange = function() {
  if(xhr.readyState == 4 && xhr.status == 200){
    console.log(xhr.response)
  }
}

xhr.open('POST', '/api/entrance/login')

const data = 'email=xxxx&password=xxxxx&verifyCode=344&key=123'

xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");

xhr.send(data)
```

## formData格式

`content-type:multipart/form-data`

一般用于传输文件、图片。

示例：

```js
let data = new FormData()
data.append('srcId', 2)
data.append('uId', uId)
data.append('envId', envId)
data.append('cntrId', cntrId)
data.append('dataJson', JSON.stringify(dataJson))

axios.post(
  url,
  data,
  {
    headers: {
      'Content-Type': 'multipart/form-data'
    }
  })
```

## json格式

`content-type:application/json`

示例：

```js
axios.post(
  url,
  {
    name: 'zbw',
    sex: 1
  },
  {
    headers: {
      'Content-Type': 'application/json'
    }
  })
```
