---
title: 使用webpack.require优化vue项目的路由
tags: [前端, Vue, VueRouter]
date: 2017-07-06
---
在使用Vue做后台管理项目的时候，项目前期,只需要维护几个路由页面即可。
在`router.js`文件中使用以下写法即可。
```JS
import Vue from 'vue';    

const Login = () => import('@/components/login/login');       
const Index = () => import('@/components/home/index');     
// balabala.......前期少量的路由                        
export default new Router({                                        
routes: [ 
    {                               
      path: '/login',                     
      name: 'Login',                   
      component: Login,                 
    },
    {                     
      path: '/home',                
      redirect: '/operation/info/list',                  
      name: 'home',
    },                      
    //balabala..........前期少量的路由     
    ])         
```
<!-- more -->
但是到了项目后期,需求越来越多,需要维护的路由也越来越多的时候,继续使用以上方法就会显得比较臃肿。
![image](/img/vue-router_1/router_1.jpeg)
看右侧栏我们不难发现这个项目,是多么的臃肿,难以维护。
当时我们的这个项目需要维护两百多个页面,维护一个`router.js`文件,在团队协作的时候同时修改此文件容易冲突。这时候我们需要一种方法来让我们不去维护一个`router.js`文件,而是自动匹配文件夹下的文件去自动生成router。
当时给出的解决方法便是使用[`require.context()`](https://webpack.js.org/guides/dependency-management/#require-context)这个由webpack提供的方法来去中心化管理前端路由。
`require.context(directory, includeSubdirs, filter)`三个参数。
- directory: 第一个参数[String],指的是要检索的文件路径, 这个参数为必传参数
- includeSubdirs: 第二个参数[Boolean],指是否检索子目录, 这个参数为可选参数, 默认为true
- filter: 第三个参数[RegExp],需要传入一个正则表达式来匹配被检索目录下符合正则的文件,这个参数为可选参数
我们了解到这个方法的参数及功能之后,便可以编写方法来自动维护**router**了。
```JS
/*eslint-disable*/
import index from './index'
import login from './login'

let router = [
  {
    path: '/login',
    name: 'login',
    component: Login,
  },
  {
    path: '/',
    name: 'index',
    component: index,
    chirldren: getRouter(),
  },
]
function getRouter() {
  let context = require.context('../pages', true, /.vue/)
  var hashArr = []
  for (let i = 0, list = context.keys(); i < list.length; i++) {
    let hash = {
      name: `${list[i].replace(/\.\//, '').replace(/\.vue/, '').split('/')[0]}`,
      path: `${list[i].replace(/\.\//, '').replace(/\.vue/, '').split('/')[0]}`,
      component: `../pages${list[i].replace(/\./, '')}`,
    }
    hashArr.push(hash)
  }
  return hashArr
}
```
当时编写了类似上面的方法,这个方法有些局限性,就是对于`page`目录底下的文件夹的命名则需要非常规范,因为这个路由会直接根据`page`下的文件夹命名来生成。
之后想到好的方法,会继续更新。。
