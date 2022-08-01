# Web API

### 密码框显示文本

```javascript

// 密码显示文本按钮事件
  document.querySelector('.bi-eye-fill').addEventListener('click', function(){
    //获得input
    let psd = document.querySelector('#input-password') 
    // 判断
    if(psd.type === 'password'){
      psd.type = 'text'
    } else {
      psd.type = 'password'
    }
  })
```

# JavaScript

### promise示例

```javascript
//获取第三方模块fs
const fs = require('fs');
// 声明构造函数Promise
let p = new Promise((resolve,rejects) => {
  fs.readFile('./a.txt','utf-8',(err,data) => {
    err ? rejects(err) : resolve(data.length);
  })
})
// 获取promise的两个实参
p.then(
  data => console.log(data),
  err => console.log(err)
)

//**********then链式写法**********
//获取第三方模块then-fs
const { readFile } = require("then-fs");

//读取文件，存入promise
let t1 = readFile("./a.txt");
let t2 = readFile("./b.txt");
let t3 = readFile("./c.txt");

//获取promise的内容
t1.then((res) => {
  console.log(res.length);
  return t2;
})
  .then((res) => {
    console.log(res.length);
    return t3;
  })
  .then((res) => {
    console.log(res.length);
  }).catch((err) => {
    //错误提示
      console.log(err)
  })

//**********async...await写法**********

//获取第三方模块then-fs
const { readFile } = require("then-fs");

//读取文件，存入promise
let t1 = readFile("./a.txt");
let t2 = readFile("./b.txt");
let t3 = readFile("./cv.txt");
//获取promise的内容
async function abc() {
  try {
    let r1 = await t1;
    let r2 = await t2;
    let r3 = await t3;
    console.log(r1.length, r2.length, r3.length);
  } catch (error) {
    //错误提示
    console.log(error);
  }
}
//执行函数
abc();

```

### axios请求

```javascript
axios(
    //请求类型
    medhod:'get',
    //请求url
    url:'http://www.itcbc.com:3006/api/getbooks',
).then((result) => {
    //获取结果
    console.log(result)
})
```

