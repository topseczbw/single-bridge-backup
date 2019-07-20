# promise

## 使用

### promise.all

1. 并行处理异步任务

   ```js
   const p1 = () => {
       return new Promise(((resolve, reject) => {
         setTimeout(() => {
           resolve(1)
         }, 1000)
       }))
   }
   
   const p2 = () => {
     return new Promise(((resolve, reject) => {
       setTimeout(() => {
         resolve(2)
       }, 3000)
     }))
   }
     
   // 3秒后返回[1, 2]，解决串行4秒后返回的问题
   Promise.all([p1(), p2()]).then((a) => console.log(a))
   ```

   


