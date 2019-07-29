# vue实战发现

1. ## provide 与 inject

   ### 是什么？

   是一种依赖注入的概念，由父级或祖先级向子级注入数据，子级接收，实现跨越无数层级组件间通讯。

   ### 为什么存在？

   为了解决跨层级组件通讯困难问题诞生。

   ### 注意！

   注入的数据并不是可响应的，这是刻意为之的，并且不能修改，会报错：避免直接改变注入的值（基本数据类型和引用数据类型的地址），因为只要提供的组件重新呈现，更改就会被覆盖。

   但是！ `如果你传入了一个可监听的对象，那么其对象的属性还是可响应的。`

    也就是说引用数据类型的属性是可以改变的，并且可以出发响应式变化，利用这一点，我们可以做很多事情。如利用入口模块向下注入核心数据对象，各子模块接收后粒度化拆分功能修改核心数据对象。

   `prop` 同理，`vue` 虽然宣扬单向数据流，但是这种类似于 `Angular` 数据双向绑定概念的操作，拆分核心对象数据到各个粒度化子组件修改，会让你的代码撸到爽到飞起。

   ### 在哪里可以使用？

   父级组件和子级组件产生了一种隐形的耦合关系，父级不知道最终将数据传给谁，子级不知道数据是从哪里获取到的，因此只建议在封闭度高的高阶组件和组件库中使用，不建议在应用代码中使用。
   
   由于传入的基本类型数据不可响应，更推荐注入对象的形式。常用于初始化项目（组件）使用的配置，比如年部学科，这些东西一般不会发生变化，子组件又不定时需要。最重要的是，不依赖于vuex，因此成为开发独立插件组件的首选。
   
   ### 示例
   
   ```javascript
   // 父级、祖先级组件
   provide() {
     return {
       List: this.List,
       type: this.type
     }
   }
   // 子组件
   inject: {
     // 类型
     type: {
       default: () => {
         return ''
       }
     },
     // 列表
     list: {
         default: () => {
           return []
         }
      }
   }
   ```
   
2. ## 动态组件

   ### 是什么？

   在某个位置，动态切换，加载不同的组件。

   ### 为什么存在？

   避免出现大量的 `v-if` , 更优雅处理业务逻辑。

   ### 注意！

   使用动态组件时，通常会重新创建不同的组件，但是当切换的是同一个组件的时候，如单选题组件和多选题组件，由于公用的是一个组件，vue内部会自动处理，不会再重新创建这个组件，不重新创建组件内的状态就无法清空，引发问题，此时需要加一个key作为唯一标识，保证组件重新创建。

   ```js
   const COMPONENT_CONFIG = {
     [QS_LOGIC_ID_MAP.SINGLE_CHOICE]: 'Choice',
     [QS_LOGIC_ID_MAP.MULTIPLE_CHOICE]: 'Choice',
     [QS_LOGIC_ID_MAP.FILL_BLANKS]: 'FillBlank',
     [QS_LOGIC_ID_MAP.RESOLVE_ANSWER]: 'ResolveQs',
     [QS_LOGIC_ID_MAP.COMPLEX]: 'Complex'
   }
   computed: {
     // 当前逻辑题型
     currentLogicId() {
       return this.mainData.logicQuesTypeId
     },
     // 当前组件
     currentComponent() {
       return COMPONENT_CONFIG[this.currentLogicId]
     }
   },
   <component :is="currentComponent" :main-data="mainData" :key="currentLogicId">
   </component>
   ```

   

   ### 在哪里可以使用？

   如根据不同的题型id，切换不同的录题组件。

   ### 示例

   ```javascript
   // currentComponent 是已注册组件的名字 或 一个组件的选项对象
   <component :is="currentComponent"></component>
   ```

3. ## watch

   watch监听值/引用地址的变化

   ### immediate

   #### 是什么？

   让所监听数据初次赋值时就触发 `handler` ，而非纯异步监听：只有改变时才出发 `handler`

   #### 为什么存在？

   在一些特定的应用场景中，我们试图监听 `prop` ，而不仅仅是本组件中的状态，但父组件对子组件 `prop` 的初次赋值我们是监听不到的，所以 `immediate` 诞生了。

   #### 在哪里可以使用？

   监听 `prop`

   #### 示例

   ```js
   props: {
     mainData: {
       type: Object
     }
   },
   watch: {
     mainData: {
       deep: true,
         // 保证父组件初始化是对mainData（null）赋值后，子组件立即响应
         immediate: true,
           handler: function() {
             this.$emit('update', this.mainData)
           }
     }
   }
   ```

4. ## 关于控制组件状态

   ### 背景

   我们经常说**粒度化**，指将**复杂的功能拆解到不同子模块处理。**

   经过使用vue了一段时间后，我们发现，由于父组件prop传进来的**值/引用地址**在**无法在子组件中直接改变**，于是我们会经常使用prop去初始化子组件中的状态/data，然后去**随心所欲**操作这份数据。

   有**两种方式**，可以实现以上目标，一是直接在data中使用prop赋予默认值，二是在created钩子中，有一个初始化状态方法 `initStatus(slefData)` 。

   我**更推荐后者**，因为也许有一天我们会遇到**父组件需要重置子组件内部状态**的场景，比如**换一题**这样的功能。

   **换一题功能的两种解决方案**

   1. 深度监听试题数据，试题数据发生变化后，重新调用子组件内的 `iniSelfData` 方法，自己刷新自己。但实际上这是消耗性能，并且不合理的操作。
   2. 考虑到**性能****，我们既不希望一个庞大的组件**重新创建**，又不想要**上一题留下的冗余的状态**。那么此时我们就可以拿到子组件实例，在获取到下一题数据后，用新的核心数据，去重新调用子组件内部 `initSatus(slefData)` 方法，达到**优雅更新子组件状态**的目的。这种操作子组件对外提供一个**接口** 即 `initStatus` 方法，**将重置组件状态的控制权交给父组件**，这样的设计更加**合情合理**。

   ### 注意

   我知道很多时候我们需要在 `created` 钩子函数中异步调接口获取数据初始化。

   但是**千万不要**写出 `async created(){}` 这样的钩子函数，因为它会将整**个创建函数变成异步函数**。

   随之带来的是内部的 `initStatus(slefData)` 函数也变成了异步，但实际上我们需要的是同步。

   ```javascript
   // 不好的写法
   async created() {
     // 初始化组件内部，一些由用户操作而改变的状态
     initStatus()
   	// 初始化数据
     const data = await requestFn()
     handlerData(data)
   }
   
   // 好的写法
   async getData() {
     return await requestFn()
   }
   created() {
     // 初始化组件内部，一些由用户操作而改变的状态
     initStatus()
   	// 初始化数据
     const data = await getData()
     handlerData(data)
   }
   ```

   ### 技巧
   
   **重置组件状态**：`Object.assign(this.$data, this.$options.data())`

5. 





