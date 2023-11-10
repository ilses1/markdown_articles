# 0.前言

> 写这篇文章的主要原因是,看到很多手写evntbus的文章和发布npm包的文章,
>
> 但是没有把两者结合在一起的,于是想要写一篇放在一起的文章,这样就实现了一个完整的写工具,到发布工具,到使用工具的整个流程.

本文最后实现的发布包链接如下:<br>
https://www.npmjs.com/package/easyer-eventbus

# 1.概念

## 1.1eventbus

eventbus即事件总线,是一种利用发布/订阅者模式进行组件间通信的方式.

在EventBus的应用中，不同组件可以订阅事件，也可以发布事件。当一个事件发生时，所有订阅了这个事件的组件都会接收到这个事件，并可以做出相应的处理。

-   以下是一个简单的使用例子:

```javascript
// 发布信息的组件
import EventBus from './event-bus.js'; // 导入EventBus  
EventBus.emit('eventName', 'hello')


// 订阅信息的组件
import EventBus from './event-bus.js'; // 导入EventBus  
EventBus.on('eventName', (data) => {
    console.log(data)
})

```

## 1.2npm包

npm相信大家都不会陌生,而且是非常熟悉,我们项目下载依赖和运行命令时会用到,类似Python的pip命令.

npm全称是*Node Package Manager*即node包管理,是nodejs默认包管理工具,我们安装nodejs时就会同时安装npm.

[npm官网有着丰富的软件包可以满足你的大部分需求](https://www.npmjs.com/)<https://www.npmjs.com>


![image-20231013100628192.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/fb9002625f04441396c434940c7dd7cb~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

但是如果我们发现没有合适的软件包,于是自己实现了一个功能,想要发布到npm网站上,以便自己以后下载或者给其他开发者使用呢?

那么怎么实现npm发布包呢?别急下面我来教会你

![image-20231013101350431.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/3908d556083d457499908e031bfc0447~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

# 2.实现

## 2.1 手写eventbus

eventbus的基本api有on(订阅),emit(发布),once(触发一次订阅后销毁),off(移除回调监听),reset(清除所有订阅)

是不是感觉有点多,有点望而却步?别怕,这里我们先实现核心的订阅on和发布emit事件


### 2.1.1先搭个框架

首先明确我们的需求:

![image-20231013105256403.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/5c7e5668806a41f29e16c3191eff693d~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

-   1).实现一个eventbus

-   2).实现eventbus的发布事件emit
-   3).实现eventbus的订阅事件on

明确需求后,我们最好先打个框架,就像写作文先写大纲后面写起来就不慌不忙,心中有数一样,写需求先写个框架,也让咱们程序员心中有底,更好完成任务 ~~(方便更好摸鱼,这个划掉)~~

首先新建一个EventBus函数,再在EventBus函数内部,新建一个on函数和一个emit函数,然后在EventBus函数最后返回on和emit函数

```JavaScript
function EventBus() {

    function on() {
    }

    function emit() {
    }

    return {
        on, emit
    }
}
```

### 2.1.2实现eventbus中的订阅事件on

这里我们先要实现的是eventbus中的订阅事件on,

> 诶这里可能有同学就要问了,为什么不先先写发布事件emit呢?实际使用不是我们先emit一个事件,然后再使用on来接收事件吗?按顺序不是应该先写emit事件吗?

![image-20231013110400550.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/6147fd0e22cb4984b5f2bb09f77f36ee~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

其实没啥为什么,先写emit也可以,硬要解释的话,可以这样理解:

实际触发事件时,先触发emit事件,然后调用对应on事件的回调函数,在emit事件之前,我们要进行订阅on,告诉eventbus订阅对应的事件和触发的回调,这样emit事件触发时,才能通知对应的事件和回调.

还是难以理解吗?我来举个外卖的栗子:

> 我现在很饿,想点一份华莱士蜜汁手扒鸡~~昨天刚吃~~,我打开美团外卖,
>
> 下单(**菜品**:蜜汁手扒鸡,**地址**:翻斗花园7栋1208),
>
> 这就相当于订阅事件on,**事件名**是蜜汁手扒鸡,**回调**是送到翻斗花园7栋2208,
>
> 等到店家的**蜜汁手扒鸡做好**后,交给骑手,骑手**查看该订单对应的地址**送货上门,
>
> 这就相当于**发布事件emit**,蜜汁手扒鸡做好**触发emit事件**,然后骑手查询该事件(蜜汁手扒鸡)对应的**回调**(送货地址),**执行回调**(送货上门)

简单来说就是先下单后配送,不下单想白嫖吗?

![image-20231013112350461.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/b8912167350c44c6b4a475b71d37eec8~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

具体实现如下:

-   1).建立一个事件池,存储键值对:事件名eventName和对应的回调数组callbacks(因为同一个事件,触发的回调有很多,所以放入数组中)
-   2).订阅事件on,传入两个参数,分别是事件名eventName和回调函数callback,


-   3).根据传入事件名eventName查询是否已经存在该事件的回调数组,没有则新建回调数组
-   4).把传入的回调函数callback加入上一步获取的回调数组callbacks
-   5).更新事件池,即在事件池中存入事件和新的回调数组

```JavaScript
// 事件池
const events = new Map();
function EventBus() {
    // 订阅事件
    function on(eventName, callback) {
        // 1.获取对应事件回调数组,没有则新建
        const callbacks = events.get(eventName) || new Set();
        // 2.把新回调加入回调数组
        callbacks.add(callback);
        // 3.更新事件池
        events.set(eventName, callbacks);
    }

    function emit() {
    }
    
    return {
        on, emit
    }
}
```

### 2.1.3实现eventbus中的发布事件emit

emit发布事件就比较简单了,传入事件名和参数,调用对应的回调即可

具体实现如下:

-   1).emit发布事件,传入事件名和参数
-   2).根据事件名获取对应事件回调数组


-   3).数组为空则抛出错误,数组存在则执行数组中每个回调并传入参数

```javascript
const events = new Map();
function EventBus() {
    function on(eventName, callback) {
        const callbacks = events.get(eventName) || new Set();
        callbacks.add(callback);
        events.set(eventName, callbacks);
    }
    // 发布事件
    function emit(eventName, payload) {
        // 1.获取对应事件回调数组
        const a = events.get(eventName);
        // 2.数组为空则抛出错误,数组存在则执行数组中每个回调并传入参数
        if (!a) throw new Error('事件不存在');
        a.forEach(fn => fn(payload));
    }

    return {
        on, emit
    }
}
```

### 2.1.4 全部代码

```javascript
const events = new Map();
function EventBus() {
    
    function on(eventName, callback) {
        const callbacks = events.get(eventName) || new Set();
        callbacks.add(callback);
        events.set(eventName, callbacks);
    }

    function emit(eventName, payload) {
        const a = events.get(eventName);
        if (!a) throw new Error('事件不存在');
        a.forEach(fn => fn(payload));
    }

    return {
        on, emit
    }
}

export default EventBus();
```

## 2.2 发布npm包

### 2.2.1 注册npm账号

注册:

<https://www.npmjs.com/signup>

登录:

<https://www.npmjs.com/login>


![image-20231013140230947.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/2d85928311df44b0835652db721fdc59~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

登录成功如下:



![image-20231013141154841.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/0f02b4f7bc2643c2bd6ee7525ac16c75~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

### 2.2.2 建立发包文件夹

-   1).新建一个文件夹easyer-eventbus


![image-20231013143622357.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/89babb89cafe4ef4a6520eda6ebae849~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)



-   2).使用vscode打开该文件夹在终端执行`npm init` 初始化package.json

基本往下按回车即可,主要是设置包名,版本号,描述,入口文件,关键词,执行命令,git地址,作者,协议等

```javascript
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help init` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (easyer-eventbus)
version: (1.0.0)
description:
entry point: (index.js)
test command:
git repository:
keywords:
author: fwy
license: (ISC)
About to write to E:\桌面\easyer-eventbus\package.json:

{
  "name": "easyer-eventbus",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo "Error: no test specified" && exit 1"
  },
  "author": "fwy",
  "license": "ISC"
}
```

-   3). 在package.json同级目录下新建index.js把步骤2.1.4 的全部代码复制到这里,这个就是我们的入口文件


![image-20231013142458607.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/f874e3f267b648228c3f28d4bb65a230~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

### 2.2.3 发布npm包

-   1).npm确认为npm原始源

如果是淘宝源或其他源改为npm源

```powershell
// 查看npm镜像源地址
npm config get registry

// 设置npm默认源
npm config set registry https://registry.npmjs.org/
```

-   2).在命令行上登录npm

```powershell
npm login
```

按步骤输入用户名,密码和邮箱的一次性密码

-   注意:
-   在这里输入密码不会显示出来,这是没问题的,输入完点回车即可
-   邮箱的一次性密码需要登录注册npm时的邮箱查看

登录成功如下:


![image-20231013142749410.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/3bd52b5b3ba348b8b2e7d629d331e2e2~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

-   3).npm发布

```
npm publish
```

-   注意:发布前请到npm官网查询是否已经有相同名字的包,如果重名则发布不成功,改一个没被使用的包名即可

发布成功如下:


![image-20231013143339924.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/2f2f3ffd479b44c8ad93358e8c2505f0~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

# 3.查询及测试

## 3.1 到官网查询刚才发布的包

<https://www.npmjs.com/>

查询成功!!!

![image-20231013143838964.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/2eecfa10558345639841900f8f27e9ac~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

> 恭喜恭喜!!!到这里你成功学会了简单手写eventbus和发布npm包了,从此你就踏上了自己造轮子的征程,一颗前端新星正在冉冉升起


![image-20231013144220795.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/33b1bdc509d345f98ee46ca05a314bda~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

如果你还有精力的话可以继续往下看

## 3.2 测试使用

发布成功后,还需要测试一下,我们的包能否正常下载和使用:

### 3.2.1.能否使用npm i 下载

-   打开之前的一个前端项目,随意打开一个即可,有package.json即可

先切换为淘宝镜像源

`npm config set registry https://registry.npm.taobao.org/`

执行下载命令

`npm i easyer-eventbus`

下载成功如下:

![image-20231013145315917.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/d7289973726b41c689723a6573eb6507~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

查看package.json


![image-20231013145327016.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/a2d74299ea2c4c38b1cebc4ced722ce8~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

### 3.2.2.eventbus能否使用

我这里直接使用我的之前的vue项目来测试

父组件app.vue

点击按钮发布信息

```javascript
<script setup lang="ts">
import test from "./pages/test.vue";
// @ts-ignore
import eventbus from "easyer-eventbus"
const emit = () => {
    eventbus.emit("test", "hello world !!")
}
</script>

<template>
    <test></test>
    <h1 @click=emit>发布信息</h1>
</template>

<style scoped></style>
```

子组件test.vue

在父组件点击按钮后text内容变化

```javascript
<script setup lang="ts">
// @ts-ignore
import eventbus from "easyer-eventbus"
import { ref } from "vue";
const text = ref('test')
eventbus.on("test", (data) => {
    console.log(data);
    text.value = data
})
</script>

<template>
    <h1> {{ text }}</h1>
</template>

<style scoped></style>
```

> 初始状态:



![image-20231013152401738.png](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/66f8f8def9bf4b0d80319c39b31a8712~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

> 预期状态:点击发布信息后如果 test 文字变为 hello world !! ,那么eventbus就是能够正常使用的

![动画.gif](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/340b09251d9247af84057e046b558bee~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

-   打包后的eventbus可以正常使用

# 4.总结

本文实现了两个模块,一个是手写简单eventbus,另一个是发布npm包.并进行了下载测试.