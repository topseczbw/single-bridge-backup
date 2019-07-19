# DIY录题服务  
本项目是放置于题库系统中的一个页面。

为提高非专业录排老师录题效率诞生。

在Paper、课件等其他外部系统中调用。

## 开发注意  
项目基于 `vue-cli-preset` 搭建，需提供 `vue-cli 3.0` 技术支持。

**本地开发环境**主要分为 **index** 和 **example** 两个路由页面。 

+ index页面（项目入口页面）用于功能源码开发，具备 `vue-devtools` 开发工具便于调试vue应用。

   开发环境下，访问index页面（主入口页面）调试源码时需要对 `/src/init.js` 文件中的 `init` 方法参照如下步骤修改:，3步：

   1. 解除【标记1】下setDevConfig方法注释(如果处于被注释状态)，并注释【标记2】下setProConfig方法
   2. 配置自定义参数
   3. 由于生产环境存在对老师年部科学权限控制，必须保证如下setDevConfig方法中参数对象的年部学科ID与根目录下vue.config.js中 pluginOptions.loginInfo.dev.loginUrl 地址中的年部学科参数保持一致，重新启动node服务：npm run serve

+ example页面用于模拟线上生产环境，模拟 *调用方* 向DIY录题服务传入必要的参数进行初始化，页面分为如下两部分：左方是对 *调用方* 生产环境中传入的参数配置。右方是模拟生产环境中，以iframe的形式将 *dev页面* 嵌入，便于对功能进行仿生产环境校验。路由地址（以localhost为例）: http://localhost:8080/#/example 

   开发环境下，访问example页面，进行仿生产环境校验时需要对 `/src/init.js` 文件中的 `init` 方法参照如下步骤修改：

   1. 解除【标记2】下setProConfig方法注释(如果处于被注释状态)，并注释【标记1】下setDevConfig方法

## API

+ 初始化DIY录题服务

  请求方式：iframe

  请求类型：GET

  示例：`https://devtk.aibeike.com/diy/index.html?subjectId=1&gradeGroupId=1&source=paper&logicIdList=1%2C2%2C4%2C8%2C9&queId=&pageType=0#/`

  参数配置：
  
  |     参数     |     说明     |  类型  | 示例 |
  | :----------: | :----------: | :----: | :----: |
  |    source    |  调用方唯一标识  | String | 'paper' |
  |  subjectId   |    学科id    | String | — |
  | gradeGroupId |    年部id    | String | — |
  |    queId     |    试题id    | String | — |
| logicIdList  | 逻辑题型列表 | String | '1,2,4,8,9' |
  |   pageType   |   页面标识   | String | '0' |

+ 获取数据

  请求方式：postMessage

  请求类型：message

  示例:

  ```javascript
  window.addEventListener(
    'message',
    function(e) {
      if (e.data.source === 'diy') {
        console.log('来自diy的数据')
        // e.data.queId
        // e.data.type
      }
    },
    false
  )
	```

  参数配置：

  | 参数  |   说明   |  类型  | 示例 |
  | :---: | :------: | :----: | :--: |
  | queId |  试题id  | String |  —   |
  | type  | 操作类型 | String | '1'  |

## 安装依赖
```
npm install
```

## 项目启动
```
npm run serve
```

## 项目打包
```
npm run build
```

## 代码检查
```
npm run lint
```

## 运行单元测试
```
npm run test:unit
```
