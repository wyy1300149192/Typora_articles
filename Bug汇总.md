# vue的异步错误

![image-20220808162438409](https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/image-20220808162438409.png)

`问题：`异步获取的数据，模板如果需要取对象第二层的数据会报以上错误，因为模板加载时，异步请求还未完成，对象第一层为undefined，第二层则会在undefined中找，所以会报错

`解决方案`在外层做if判断，异步数据获取后再渲染

![image-20220808162857530](https://yang-cloud-img.oss-cn-beijing.aliyuncs.com/img/image-20220808162857530.png)