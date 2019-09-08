# 文件上传阿里云oss
<!-- TOC -->

- [什么是分片上传](#什么是分片上传)
- [什么是断点续传](#什么是断点续传)
- [上传OSS步骤](#上传oss步骤)
- [删除正在上传的图片](#删除正在上传的图片)
- [常见问题](#常见问题)
  - [上传同名文件是否会覆盖](#上传同名文件是否会覆盖)
  - [移动端（钉钉微应用）上传时，出现少数机型分片上传功能失效](#移动端钉钉微应用上传时出现少数机型分片上传功能失效)
    - [解决方案](#解决方案)
    - [相关链接](#相关链接)

<!-- /TOC -->
## 什么是分片上传

在需要上传的文件较大时，可以通过`multipartUpload`接口进行分片上传。分片上传的好处是将一个大请求分成多个小请求来执行，这样当其中一些请求失败后，不需要重新上传整个文件，而只需要上传失败的分片就可以了。一般对于大于100MB的文件，建议采用分片上传的方法。

## 什么是断点续传

分片上传提供**progress**参数允许用户传递一个进度回调，在回调中SDK将当前已经**上传成功的比例**和**断点信息**作为参数**供前端使用**。为了实现断点上传，可以在上传过程中**保存断点信息**（checkpoint），**发生错误后，再将已保存的checkpoint作为参数传递给multipartUpload**，此时将从上次失败的地方继续上传。

```js
try {
      const result = await client.multipartUpload('object-name', filePath, {
        checkpoint,
        async progress(percentage, cpt) {
          checkpoint = cpt;
        },
      });
      console.log(result);
      // break if success
      break;
    } catch (e) {
      console.log(e);
}
```

上面的代码只是将checkpoint保存在变量中，如果程序崩溃的话就丢失了，用户也可以将它保存在文件中，然后在程序重启后将checkpoint信息从文件中读取出来。

## 上传OSS步骤

1. 引入**aliyun-oss-sdk.min.js**
2. 请求后端接口，根据后端接口返回的数据，创建一个OSS对象
3. 拿到**File**对象，由于在阿里云存储空间的 `backet` 里同名的文件会被覆盖掉，需要拼接一个随机的名字key传给阿里云
4. 调用OSS对象的**multipartUpload**的方法，传入【文件名：上述生成一个随机生成的key】【文件对象】【process进度回调事件】【断点信息对象】等一些信息。
5. **then**方法中表示，文件已上传OSS成功，可以拿到**文件相关的信息**，作为【前端展示】，【传递后端保存与数据库】的用途

示例：

```js

<template>
  <div class="index">
    <div class="oss_file">
      <input type="file" :id="id" @change="doUpload"/>
      <hr>
      <p>
        上传进度：{{percentage}}
      {{percentage===1?"success!":(percentage===0?'waiting...':'uploading')}}
      </p>
      <hr>
      <ul>
        <li v-for="u in urls">{{u}}</li>
      </ul>
    </div>
  </div>
</template>

doUpload () {
        const _this = this
        const urls = [];
        _this.axios.get('http://106.15.89.254:8188/alioss/getOssToken').then((result) => {
          // 这个token一般存储在后端
          const client = new OSS.Wrapper({
            // 实例所在的地理位置
            region: _this.region,
            // 填写自己阿里云账户的acessKeyId
            accessKeyId: result.data.data.AccessKeyId,
            // 填写自己阿里云账户的accessKeySecret'
            accessKeySecret: result.data.data.AccessKeySecret,
            stsToken: result.data.data.SecurityToken,
            // 在oss后台创建的bucket
            bucket: _this.bucket
          })
          // 上传进度百分比
          _this.percentage = 0;
          // 获取input元素
          const files = document.getElementById(_this.id)
          if (files.files) {
            const fileLen = document.getElementById(_this.id).files
            let resultUpload = ''
            for (let i = 0; i < fileLen.length; i++) {
              const file = fileLen[i]
              // 随机命名
              let random_name = this.random_string(6) + '_' + new Date().getTime() + '.' + file.name.split('.').pop()
              // multipartUpload 阿里云分片上传
              client.multipartUpload(random_name, file, {
                progress: function* (percentage, cpt) {
                  // 上传进度
                  _this.percentage = percentage
                }
              }).then(function (res) {
        let urlValue = res.res.requestUrls[0]
        // 有时上传阿里会加uploadId=。。。 后缀，导致图片无法显示，用正则截取掉
        const reg = /\?uploadId=[0-9|A-Z|a-z]{32}/g
        urlValue = urlValue.replace(reg, '')
        _self.setFile(realId, type, imgList, 'url', urlValue) // 设置文件网址
        // 上传成功之后清除value，防止change事件上传相同的文件不能再次上传
        document.getElementById('file').value = ''
      }).catch(function (e) {
        if (e.name !== 'cancel') {
          ddAlert('提交失败，请检查您的网络连接')
          // 这里用1来设置，不用''来设置，防止跟undefined冲突,用来判断显示上传失败
          _self.setFile(realId, type, imgList, 'url', 1) // 设置文件网址
        }
        console.log(e)
        // 上传失败之后清除value，防止change事件上传相同的文件不能再次上传
        document.getElementById('file').value = ''
      })
            }
          }
        })
},

    // 随机生成文件名
    random_string(len) {
        len = len || 32;
        var chars = 'ABCDEFGHJKMNPQRSTWXYZabcdefhijkmnprstwxyz2345678';
        var maxPos = chars.length;
        var pwd = '';
        for (let i = 0; i < len; i++) {
            pwd += chars.charAt(Math.floor(Math.random() * maxPos));
        }
        return pwd;
    }

  watch:{
      url(val){
        if(val){
        this.urls.push(val);
        }
      }
  }

```

## 删除正在上传的图片

```js
deleteImg (data) {
      this.deleteHandel(data, this.stopImgList)
      this.imgList.splice(data.index, 1)
    },
    deleteHandel (data, list) { // 暂停及实时更新client对象
      if (data.progress < 100) {
        list[data.index].client.cancel()
      }
      list.splice(data.index, 1)
      // 点击删除也要同时清空value值，防止暂停有时上传不了文件
      document.getElementById('file').value = ''
    },
```

## 常见问题

### 上传同名文件是否会覆盖

OSS允许上传同名文件，但是会对源文件直接执行覆盖操作

### 移动端（钉钉微应用）上传时，出现少数机型分片上传功能失效

#### 解决方案

实例化 OSS 选项的时候传入useFetch: false选项

#### 相关链接

[h5 OSS上传 qq浏览器失败超时](https://www.jianshu.com/p/5f6d09c099b6)

[alioss-js-upload](https://github.com/taosin/alioss-js-upload/blob/2.0/public/src/components/upload.vue)
