# Vue2案例

## npm 对应 yarn 命令

![](https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/1658992332839.png)

## 解决跨域问题

### vue-cil 代理服务器

1. **修改配置文件.env.development**

```javascript
//.env.development 中的VUE_APP_BASE_API改成 /api

# just a flag
ENV = 'development'

# base api
VUE_APP_BASE_API = '/api'

```

2. **vue-config 配置*module*._exports_**

```javascript
devServer: {
    proxy: {
      // 这里的api 表示如果我们的请求地址有/api的时候,就出触发代理机制
      '/api': {
        target: 'http://ihrm.itheima.net/', // 我们要代理请求的地址
        changeOrigin: true //是否跨域
      }
    }
  },
```

## 登录模块设计流程

1. 设计**表单、登录按钮**等布局
2. 表单绑定 data 数据对象
3. 表单校验
4. 处理登录按钮事件，点击登录先进行兜底校验（可封装在 utils 中的 validate.js 中），校验通过再执行下面的：
5. 根据接口文档写好请求：写封装在 api 文件夹的 js 中
6. 随后发送请求登录（判断是否有数据需要存入 vuex，有则使用 vuex 请求）
7. 将 token 存入并持久化，持久化可使用 cookies
8. 跳转至登录后页面

## 退出登录流程

1. store 设置好 actions：
   - 清除用户信息，
   - 清除 token，
   - 跳转登录页，
   - 提示
2. 设置`退出登录`按钮事件，执行上面的 actions

## 登录未遂设置

**退出登录时**

1. 退出登录时跳转并携带当前路由

   ```js
   // 跳转并携带退出前路由
   // ?后面的redirect名字为自己定义的，=后面的参数会存到route.query中
   this.$router.replace(
     `/login?redirect=${encodeURIComponent(this.$route.fullPath)}`
   );
   ```

2. 在登录跳转判断 route.query.replace 有路径则跳转至这里，否则跳转至首页

   ```js
   // 跳转 ：判断有携带地址则跳转携带地址，否则跳转首页
   this.$router.push(this.$route.query.redirect || "/");
   ```

**token 过期退出登录时**

1. token 过期退出登录时携带当前路由

   ```js
   // 跳转并携带当前页面路径
   router.replace(
     `/login?redirect=${encodeURIComponent(router.currentRoute.fullPath)}`
   );
   ```

2. 在登录跳转判断 route.query.replace 有路径则跳转至这里，否则跳转至首页

   ```js
   // 跳转 ：判断有携带地址则跳转携带地址，否则跳转首页
   this.$router.push(this.$route.query.redirect || "/");
   ```

## 让 axios 少一层 data

```JavaScript
// 响应拦截器
instance.interceptors.response.use(function(response) {
  // 对响应数据做点什么
  const res = response.data
  return res
}, function(error) {
  // 对响应错误做点什么
  return Promise.reject(error)
})
```

## 让路由跳转有进度条

**使用 nprogress**：

**安装**

```
npm install --save nprogress
```

直接引入 js、css 或者通过 cdn 引入。

```html
<script src="nprogress.js"></script>
<link rel="stylesheet" href="nprogress.css" />
```

**使用**

直接调用 `start()`或者`done()`来控制进度条。

```javascript
NProgress.start();
NProgress.done();
```

可以通过调用 `.set(n)`来设置进度，n 是 0-1 的数字。

```javascript
NProgress.set(0.0); // Sorta same as .start()
NProgress.set(0.4);
NProgress.set(1.0); // Sorta same as .done()
```

## 页面 token 判断

**跳转路由时判断 token 是否存在**：

1. 将请求获得得 token 存入 state 中，并存入 Cookies 中，将 state 中的值设置为 cookies 存入的函数，否则刷新页面则会失效
2. 判断如果有 token：访问的是 login 页面则直接跳转到主页，不是登录也则放行

3. 如果没有 token：判断是白名单的直接放行，否则跳转至注册页

```javascript
// 路由白名单
const whiteList = ["/login"];
// 路由前置守卫
router.beforeEach(async (to, from, next) => {
  // 进度条开始
  NProgress.start();
  // 获取vuex中getters的token
  const token = store.getters.token;
  // 判断是否有token
  if (token) {
    // 有token访问的是login
    if (to.path === "/login") {
      // 跳转至主页
      next("/");
      NProgress.done();
    } else {
      // 放行
      next();
      NProgress.done();
    }
  } else {
    // 如果没有token
    // 判断访问的如果是白名单的地址
    if (whiteList.some((item) => item === to.path)) {
      // 放行
      next();
      NProgress.done();
    } else {
      // 跳转至登录页
      next("/login");
      NProgress.done();
    }
  }
});

// 路由后置守卫
router.afterEach(() => {
  // 进度条结束
  NProgress.done();
});
```

**跳转路由时判断 token 是否存在**：

1. 在响应拦截器中错误中判断错误 code 是否为 token 过期

2. 如果过期：

   - 删除 token
   - 删除用户信息
   - 跳转并携带当前页面路径

   ```js
   // 响应拦截器
   service.interceptors.response.use(
     (response) => {
       return response.data;
     },
     (error) => {
       // 如果过期：删除token 删除用户信息 跳转并携带当前页面路径
       if (
         error.response &&
         error.response.data &&
         error.response.data.code === 10002
       ) {
         // 提醒
         Message.error("登录过期，请重新登录");
         // 删除token
         store.commit("user/REMOVE_TOKEN");
         // 删除用户信息
         store.commit("user/RESET_STATE");
         // 跳转并携带当前页面路径
         router.replace(
           `/login?redirect=${encodeURIComponent(router.currentRoute.fullPath)}`
         );
       } else {
         Message.error({ message: error || "出现错误，请稍后再试" });
       }
       console.dir(error);
       return Promise.reject(error);
     }
   );
   ```

## 路由前置统一设置请求头 token

1. 将 vuex 中 state 的 token 放在 getters 中

2. 请求拦截器设置：

   ```
   // 请求拦截器
   service.interceptors.request.use(
     (config) => {
       const token = store.getters.token
       if (token) {
         config.headers.Authorization = `Bearer ${token}`
       }
       return config
     },
     (error) => {
       return Promise.reject(error)
     }
   )
   ```

## 导航栏 logo 控制

1. 是否显示设置在 src 下的 settings.js 中
2. 存入 vuex 中的 state 中
3. 在 logo 组件的 v-if 等于 vuex 中的 state

## 获取用户信息

1. 封装获取用户请求
2. 在前置路由守卫调用请求，存入 vuex 中

## 全局组件统一封装

如果希望在所有页面都要用, 那么就要`全局注册`组件，如果有很多个组件, 都要全局注册, 那么又会在 main.js 中写很多内容

1. Vue.use 可以接收一个对象, Vue.use(obj)
2. 对象中需要提供一个 install 函数
3. install 函数可以拿到参数 Vue, 且将来会在 Vue.use 时, 自动调用该 install 函数

index.js 统一引入组件

```js
import PageTools from "./PageTools";

export default {
  install(Vue) {
    Vue.component("PageTools", PageTools);
  },
};
```

入口文件注册插件（main.js）

```js
import Components from "./components";
Vue.use(Components);
```

## 封装过滤器

1. src 目录下创建`filters/index.js`文件

   ```js
   // 导入dayjs
   import dayjs from "dayjs";
   // 格式化组件过滤器
   export function formatData(value, str = "YYYY年MM月DD日") {
     return dayjs(value).format(str);
   }
   ```

2. 入口文件`main.js`全局注册过滤器

   ```js
   // 引入过滤器
   import * as filters from "@/filters";
   Object.keys(filters).forEach((key) => {
     // 注册过滤器
     Vue.filter(key, filters[key]);
   });
   ```

## Excel 导入

1. **封装 excel 导入组件(uploadExcel)**

   需要的依赖：`xlsx`

   ```vue
   <template>
     <div>
       <input
         ref="excel-upload-input"
         class="excel-upload-input"
         type="file"
         accept=".xlsx, .xls"
         @change="handleClick"
       />
       <div class="box">
         <div class="left">
           <el-button
             :loading="loading"
             style="margin-left: 16px"
             size="mini"
             type="primary"
             @click="handleUpload"
           >
             点击上传
           </el-button>
           <div class="text">(推荐下载模板文件，填写后上传)</div>
           <div class="text">点击查看文件上传要求</div>
         </div>
         <div
           class="rigth"
           @drop="handleDrop"
           @dragover="handleDragover"
           @dragenter="handleDragover"
           @click="handleUpload"
         >
           <div class="el-icon-upload" />
           <div>将文件拖入上传</div>
         </div>
       </div>
     </div>
   </template>

   <script>
   import * as XLSX from "xlsx";

   export default {
     props: {
       beforeUpload: Function, // eslint-disable-line
       onSuccess: Function, // eslint-disable-line
     },
     data() {
       return {
         loading: false,
         excelData: {
           header: null,
           results: null,
         },
       };
     },
     methods: {
       generateData({ header, results }) {
         this.excelData.header = header;
         this.excelData.results = results;
         this.onSuccess && this.onSuccess(this.excelData);
       },
       handleDrop(e) {
         e.stopPropagation();
         e.preventDefault();
         if (this.loading) return;
         const files = e.dataTransfer.files;
         if (files.length !== 1) {
           this.$message.error("Only support uploading one file!");
           return;
         }
         const rawFile = files[0]; // only use files[0]

         if (!this.isExcel(rawFile)) {
           this.$message.error(
             "Only supports upload .xlsx, .xls, .csv suffix files"
           );
           return false;
         }
         this.upload(rawFile);
         e.stopPropagation();
         e.preventDefault();
       },
       handleDragover(e) {
         e.stopPropagation();
         e.preventDefault();
         e.dataTransfer.dropEffect = "copy";
       },
       handleUpload() {
         this.$refs["excel-upload-input"].click();
       },
       handleClick(e) {
         const files = e.target.files;
         const rawFile = files[0]; // only use files[0]
         if (!rawFile) return;
         this.upload(rawFile);
       },
       upload(rawFile) {
         this.$refs["excel-upload-input"].value = null; // fix can't select the same excel

         if (!this.beforeUpload) {
           this.readerData(rawFile);
           return;
         }
         const before = this.beforeUpload(rawFile);
         if (before) {
           this.readerData(rawFile);
         }
       },
       readerData(rawFile) {
         this.loading = true;
         return new Promise((resolve, reject) => {
           const reader = new FileReader();
           reader.onload = (e) => {
             const data = e.target.result;
             const workbook = XLSX.read(data, { type: "array" });
             const firstSheetName = workbook.SheetNames[0];
             const worksheet = workbook.Sheets[firstSheetName];
             const header = this.getHeaderRow(worksheet);
             const results = XLSX.utils.sheet_to_json(worksheet);
             this.generateData({ header, results });
             this.loading = false;
             resolve();
           };
           reader.readAsArrayBuffer(rawFile);
         });
       },
       getHeaderRow(sheet) {
         const headers = [];
         const range = XLSX.utils.decode_range(sheet["!ref"]);
         let C;
         const R = range.s.r;
         /* start in the first row */
         for (C = range.s.c; C <= range.e.c; ++C) {
           /* walk every column in the range */
           const cell = sheet[XLSX.utils.encode_cell({ c: C, r: R })];
           /* find the cell in the first row */
           let hdr = "UNKNOWN " + C; // <-- replace with your desired default
           if (cell && cell.t) hdr = XLSX.utils.format_cell(cell);
           headers.push(hdr);
         }
         return headers;
       },
       isExcel(file) {
         return /\.(xlsx|xls|csv)$/.test(file.name);
       },
     },
   };
   </script>

   <style scoped>
   .box {
     text-align: center;
   }
   .excel-upload-input {
     display: none;
     z-index: -9999;
   }
   .rigth {
     border: 2px dashed #bbb;
     width: 400px;
     height: 180px;
     font-size: 24px;
     border-radius: 5px;
     text-align: center;
     color: #bbb;
     display: inline-block;
     border-left: 0;
   }

   .left {
     vertical-align: top;
     padding: 50px;
     border: 2px dashed #bbb;
     width: 400px;
     height: 180px;
     font-size: 24px;
     border-radius: 5px;
     text-align: center;
     color: #bbb;
     display: inline-block;
   }
   .el-icon-upload {
     font-size: 100px;
   }
   .left .text {
     font-size: 12px;
     color: #000;
     margin-top: 5px;
   }
   </style>
   ```

2. **上述组件接受两个函数**

   **beforeUpload** 导入前的回调 有一个实参: 文件对象

   - beforeUpload 需要**返回 true**才可继续导入成功的回调

   **onSuccess** 导入成功后的回调 有一个实参：一个对象，对象有两个属性`results`, `header`\*\*

   - results 代表 **表体数据**
   - header 代表 **表头数据**

   ```vue
   <template>
     <div class="app-container">
       <el-card>
         <div class="title">员工导入</div>
         <el-alert
           type="warning"
           :closable="false"
           show-icon
           title="每次导入仅可添加1000名员工，姓名、手机、入职时间、聘用形式为必填项"
         />
         <div class="import">
           <!-- 导入组件 -->
           <upload-excel-component
             :on-success="handleSuccess"
             :before-upload="beforeUpload"
           />
         </div>
       		</el-card>
     </div>
   </template>

   <script>
   import UploadExcelComponent from "@/components/UploadExcel/index.vue";

   export default {
     name: "UploadExcel",
     components: { UploadExcelComponent },
     data() {
       return {};
     },
     methods: {
       // 导入成功回调
       handleSuccess({ results, header }) {
         console.log(results);
         console.log(header);
       },
       // 导入前的回调，可用于判断文件大小、类型等
       beforeUpload(file) {
         console.log(file);
       },
     },
   };
   </script>

   <style lang="scss" scoped>
   .title {
     text-align: center;
     font-size: 20px;
     font-weight: 900;
     margin-bottom: 15px;
   }
   .import {
     margin-top: 100px;
   }
   .app-container {
     background-color: #f0f2f4;
   }
   </style>
   ```

3. 最后 header 获取的是表头的数组

   ```js
   ["入职日期", "手机号", "姓名", "转正日期", "工号"];
   ```

4. results 获取的表体 每一行一个对象的数组

   ```js
   [{…}, {…}]
   ```

5. Excel 日期格式化

   ```javascript
    formatDate(numb, format) {
         const time = new Date((numb - 25567) * 24 * 3600000 - 5 * 60 * 1000 - 43 * 1000 - 24 * 3600000 - 8 * 3600000)
         const year = time.getFullYear() + ''
         const month = time.getMonth() + 1 + ''
         const date = time.getDate() + ''
         if (format && format.length === 1) {
           return year + format + (month < 10 ? '0' + month : month) + format + (date < 10 ? '0' + date : date)
         }
         return year + (month < 10 ? '0' + month : month) + (date < 10 ? '0' + date : date)
       }
   ```

# Vue3案例

## 目录解析

![01](https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/01.png)

## Vuex持久化

利用`vuex-persistedstate`插件实现vuex数据持久化，将数据存入localStorage中

**实现步骤**

1. **安装**

   ```cmd
   npm i vuex-persistedstate
   ```

2. **vuex的plugins配置中使用createPersistedstate函数**

   * key： localStorage键名

   * paths： 需要存入state中的那些数据

   ```js
   import { createStore } from 'vuex'
   import createPersistedstate from 'vuex-persistedstate'
   
   export default createStore({
     plugins: [
       createPersistedstate({
         // key localStorage键名
         key: 'erabbit-client-pc-store',
         // paths 需要存入state中的那些数据
         paths: ['user', 'cart']
       })
     ]
   })
   ```

## Logger Plugin(vue3的vuex调试)

在vue3中vue2中的调试工具已不支持当前查看，需要我们使用`Logger Plugin`调试

Logger Plugin为vuex内置的一个模块，只需要引入并且注册为插件即可

```js
import { createStore, createLogger } from 'vuex'
export default createStore({
  plugins: [
    createLogger()
  ]
})
```

安装好这个插件后，每次触发action函数和mutation函数，都自动在控制台打印出提交记录与详细信息，包括`名称` `参数` `修改前后的state数据`

![image-20220801175511077](https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/image-20220801175511077.png)

## style的自动化导入

> 使用vue-cli的style-resoures-loader 插件完成自动注入到每个vue组件的style标签，避免在每个样式文件中手动的@import导入

1） 执行命令`vue add style-resources-loader`

2）安装完成后`vue.config.js`中自动添加配置

```js
module.exports = {
  pluginOptions: {
    'style-resources-loader': {
      preProcessor: 'less',
      patterns: []
    }
  }
}
```

2）在patterns数组中配置需要导入的style

```js
patterns: [
        // 配置哪些文件需要自动导入
path.join(__dirname,'./src/styles/variables.less')
      ]
```



## 全局组件封装

components/index.js 统一注册组件

```js
// 引入组件
import Skeleton from './Skeleton'

// 导出install方法 注册组件
export default {
  install (app) {
    app.component(Skeleton.name, Skeleton)
  }
}
```

main.js 入口文件`vue实例`应用组件

```js
import componentPlugin from '@/components'
// use应用组件
createApp(App).use(store).use(router).use(componentPlugin).mount('#app')
```

## 全局自定义指令封装

`@/directives/index.js`

```js
export default {
  install (app) {
    app.directive('指令名', {
      // el为绑定该指令的dom
      // binding可以获得该指令的值
      mounted (el, binding) {
		// 执行
      }
    })
  }
}

```

vue注册

```js
import directivePlugin from '@/directives'

createApp(App).use(directivePlugin)
```



## vueuse/core工具库

> VueUse是基于[Composion API](https://v3.vuejs.org/guide/composition-api-introduction.html)的实用程序函数的集合。

**安装**

```cmd
npm i @vueuse/core
```

### 监听滚动获得滚动距离

利用`useWindowScroll`方法监听滚动距离

```html
<script>
import HeaderNav from './header-nav'
import { useWindowScroll } from '@vueuse/core'
export default {
  name: 'AppHeaderSticky',
  components: { HeaderNav },
  setup () {
    // y表示具体顶部的滚动距离 会动态更新
    const { y } = useWindowScroll()
    return { y }
  }
}
</script>
```

### ⭐组件数据懒加载

> 电商项目的核心优化手段：组件数据懒加载
>
> 说明：电商项目大多数据很多，默认就加载一个页面的所有数据，必然会发送很多不需要的请求
>
> 解决方案：`等所在模块进入 可视区 再获取数据`

**方案**：

我们可以使用 `@vueuse/core` 中的 `useIntersectionObserver`方法 来实现监听组件进入可视区域行为

**实现**：

封装公共方法

判断处于可视区域时才执行请求函数

利用stop函数避免重复执行请求函数

```js
import { ref } from 'vue'
import { useIntersectionObserver } from '@vueuse/core'

export function useObserver (apiFunc) {
  // target为绑定的dom节点
  const target = ref(null)
  const { stop } = useIntersectionObserver(
    target,
    // isIntersecting为是否处于可视区域 返回一个布尔值
    ([{ isIntersecting }], observerElement) => {
      // 判断如果处于可视区域执行api函数
      if (isIntersecting) {
        apiFunc()
        // 停止监听
        stop()
      }
    }
  )
  // 返回一个target
  return target
}

```

### ⭐图片懒加载

> 电商类项目图片非常多，可以做一下图片的懒加载

**方案**：利用自定义指令与vueuse/core工具库 监测在可视距离后再给img的src赋值

```js
import { useIntersectionObserver } from '@vueuse/core'

export default {
  install (app) {
    //定义自定义指令名 imgLazy
    app.directive('imgLazy', {
      mounted (el, binding) {
        const { stop } = useIntersectionObserver(
          el,
          ([{ isIntersecting }], observerElement) => {
            if (isIntersecting) {
              // 监测在可视区域后再将img标签src赋值
              el.src = binding.value
              stop()
            }
          }
        )
      }
    })
  }
}
```

**使用**

```html
<img v-imgLazy='item.picture' alt="">
```



## 骨架组件业务使用

> 当页面数据还未获取到时，利用骨架组件进行占位，提升用户体验

1）定义公共骨架组件

组件接收三个值： 宽 、高 、背景色

```vue
<template>
  <!-- 决定组件的宽高 -->
  <div
    class="xtx-skeleton shan"
    :style="{ width: width + 'px', height: height + 'px' }"
  >
    <!-- 决定组件的背景色 -->
    <div class="block" :style="{ backgroundColor: bg }"></div>
  </div>
</template>
<script>
export default {
  name: 'XtxSkeleton',
  props: {
    // 宽度定制
    width: {
      type: Number,
      default: 100
    },
    // 高度定制
    height: {
      type: Number,
      default: 60
    },
    // 背景颜色定制
    bg: {
      type: String,
      default: '#ccc'
    }
  }
}
</script>
<style scoped lang="less">
.xtx-skeleton {
  display: inline-block;
  position: relative;
  overflow: hidden;
  vertical-align: middle;
  .block {
    width: 100%;
    height: 100%;
    border-radius: 2px;
  }
}
.shan {
  &::after {
    content: "";
    position: absolute;
    animation: shan 1.5s ease 0s infinite;
    top: 0;
    width: 50%;
    height: 100%;
    background: linear-gradient(
      to left,
      rgba(255, 255, 255, 0) 0,
      rgba(255, 255, 255, 0.3) 50%,
      rgba(255, 255, 255, 0) 100%
    );
    transform: skewX(-45deg);
  }
}
@keyframes shan {
  0% {
    left: -100%;
  }
  100% {
    left: 120%;
  }
}
</style>

```

2）在数据处使用v-if进行判断数据未请求到时展示骨架

```vue
<template v-if="list.length > 0">
      <li v-for="item in list" :key="item.id">
        <RouterLink :to="'/category/' + item.id">{{ item.name }}</RouterLink>
      </li>
</template>
    
<template v-else>
      <li v-for="i in 9" :key="i">
        <Skeleton
          :width="60"
          :height="32"
          style="margin-right: 5px"
          bg="rgba(0,0,0,0.5)"
        />
      </li>
```

## 面包屑组件封装

**封装两个组件**：面包屑组件与面包屑项目组件

`面包屑组件`

- 接受一个值：面包屑的分隔符
- 有一个默认插槽
- 将分隔符传出，可供项目组件使用

```vue
<template>
  <div class="xtx-bread">
    <slot />
  </div>
</template>

<script>
// 分隔符数据是位于Bread组件中 而对于分隔符数据的使用是在底层的组件中使用
// provide/inject
import { provide } from 'vue'
export default {
  name: 'XtxBread',
  props: {
    separator: {
      type: String,
      default: ''
    }
  },
  setup (props) {
    // 为底层组件提供数据
    provide('separator', props.separator)
  }
}
</script>

<style scoped lang="less">
.xtx-bread {
  display: flex;
  padding: 25px 10px;
  &-item {
    a {
      color: #666;
      transition: all 0.4s;
      &:hover {
        color: @xtxColor;
      }
    }
  }
  i {
    font-size: 12px;
    margin-left: 5px;
    margin-right: 5px;
    line-height: 22px;
  }
}
</style>
```

`面包屑项目组件`

- 接收一个值：to 用作路由跳转

- 有一个默认插槽放文字
- 判断如果to存在渲染路由链接标签，否则渲染一个span标签
- 接收面包屑组件的分隔符，如没有用默认的

```vue
<template>
  <div class="xtx-bread-item">
    <!--
      如果to存在 有值 我们就渲染一个router-link标签
      如果to不存在  那就渲染一个span标签
     -->
    <router-link v-if="to" :to="to"><slot /></router-link>
    <span v-else><slot /></span>
    <!-- 分隔符 -->
    <i v-if="separator">{{ separator }}</i>
    <i v-else class="iconfont icon-angle-right"></i>
  </div>
</template>

<script>
import { inject } from 'vue'
export default {
  name: 'XtxBreadItem',
  props: {
    to: {
      type: String
    }
  },
  setup () {
    const separator = inject('separator')
    return {
      separator
    }
  }
}
</script>
<style lang="less" scoped>
.xtx-bread-item {
  i {
    margin: 0 6px;
    font-size: 10px;
  }
  // 最后一个i隐藏
  &:nth-last-of-type(1) {
    i {
      display: none;
    }
  }
}
</style>

```

**应用**：

```vue
<XtxBread>
  <XtxBreadItem to="/">首页</XtxBreadItem>
  <XtxBreadItem>美食</XtxBreadItem>
</XtxBread>
```

