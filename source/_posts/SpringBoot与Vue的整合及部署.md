---
title: SpringBoot与Vue的整合及部署
date: 2019-05-12 13:06:37
tags:
- SpringBoot
- Vue
---

# SpringBoot与Vue的整合实现前后端分离

## vue项目创建

近期工作的项目中采用了Spring和Vue整合达到前后端完全分离的目的。vue使用了elementUI的UI框架，一边抄官方文档，一边修修改改，很方便上手。

😊这里学习整理一下，准备用到自己的毕业设计里….

###  创建vue项目


    vue init webpack 



（`注意的是，谨慎使用eslint这个工具`）

### 引入以及配置依赖包

若使用elementUI组件（需要安装依赖npm install --save element-ui）:

1. 在src文件夹main.js文件中添加依赖以及引用

        /* elementUI组件引用 */
        import ElementUI from 'element-ui';
        import '../node_modules/element-ui/lib/theme-chalk/index.css';
        /* elementUI组件调用 */
        Vue.use(ElementUI)



2. 另外还可以加上axios的依赖

        /* axios引用 */
        import axios from 'axios';
        /* axios掉用 */
        Vue.prototype.$axios = axios
        /* 设置header头，表示请求是ajax请求 */（这一行虽然写了注释，但是不懂，不知道可不可以不写）
        axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';



### 编写实例测试

在components中编写实例:


    <template>  
    <div class="hello">  
        <h1>{{ msg }}</h1>  
        <h2>Essential Links</h2>  
        <el-button>默认按钮</el-button>  
        <el-button type="primary">主要按钮</el-button>  
        <el-button type="text">文字按钮</el-button>  
        <div>
        <el-select v-model="value" placeholder="请选择">
            <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value">
            </el-option>
        </el-select>
        </div>
    </div>  
    </template>  

    <script>  

    export default {  
    name: 'hello',  
    data () {  
        return {  
        msg: '我是第二页',
        options: [{
            value: '选项1',
            label: '黄金糕'
            }, {
            value: '选项2',
            label: '双皮奶'
            }, {
            value: '选项3',
            label: '蚵仔煎'
            }, {
            value: '选项4',
            label: '龙须面'
            }, {
            value: '选项5',
            label: '北京烤鸭'
            }],
            value:"", 
        }  
    },
    created: function(){
        this.login();
    },
    methods: {
        login(){
        console.log("login");
        this.$axios.get(_const.baseUrl + "/login")
        .then(function(res){
            console.log(res.data);
        })
        .catch(function(error){
            console.log(error);
        });
        },
    }, 
    }  
    </script>  





### 配置全局变量

axios的地址可以在全局变量中定义然后使用，具体操作可以在static文件下新建一个全局变量文件：


    const sysConfig = {
        baseIp: '127.0.0.1',
        basePort: '8888',
        protocol: 'http://',
        baseUrl: 'http://127.0.0.1:8888/blog',
    }
    let bindToGlobal = (obj, key) => {
        if (typeof window[key] === 'undefined') {
            window[key] = {};
        }

        for (let i in obj) {
            window[key][i] = obj[i]
        }
    }
    bindToGlobal(sysConfig,'_const')






### 附！项目文件结构

![Eh1EHe.png](https://s2.ax1x.com/2019/05/12/Eh1EHe.png)

### 运行启动前端项目

先安装依赖：

    npm install

后运行项目:

    npm run dev

## SpringBoot创建

### 新建springboot项目

用idea新建一个springboot项目，打包形式首先选择成war包（若一开始选择的是jar包，后期通过修改启动类以及pom文件改成war打包形式，具体参见"springboot项目打包成war记录"）

### 编写实例测试

写一个controller类作测试:

    package com.csidez.blog.controller;

    import org.springframework.web.bind.annotation.*;

    //允许跨域访问
    @CrossOrigin
    @RestController
    public class testController {

        @GetMapping(value = "/login")
        public String login(){
            return "Success";
        }
    }


### 运行启动后端项目

## 整合及部署

### 端口问题

因为两个都是跑在8080端口上的，跑了一个另一个会冲突，所以本地测试的时候我修改springboot的tomcat的端口，可以在properties文件中添加`server.port=8888`这一行将端口修改到8888上面了

### 部署问题

war包是在tomcat的webapps文件下解压开的，jar -xvf *.war所以会生成相应的名字，这样springboot访问时，连接中也需要加上该文件名。
因此，在设置前端axios的地址中，定义时需要注意url地址。
vue使用cnpm run build打包生成的dist文件，将dist里的两个文件直接黏贴到解压开的war中，这样出现另一个问题，无法打开vue的页面，需要修改在config文件夹中的index.js文件。将

    assetsPublicPath: '/'

修改为


    assetsPublicPath: './'

或者也可以改为


    assetsPublicPath: '/war包文件夹名字/',


注意需要改两处

### 跨域问题

由于我的部署方式是将vue项目打的包直接丢进springboot的war包解压开的文件夹内，未见此问题。如果报一下错误

    No 'Access-Control-Allow-Origin' header is present on the requested resource.

应该是前面部署整合时候处理URL的问题，没有处理好。

