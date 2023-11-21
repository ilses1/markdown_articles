npm run 做了什么？可不可以不用run？



# 1.前言

最近敲了一个后端nodejs+MongoDB+express，前端vue3+vite的项目。

然后我发现前后端启动的命令不同，这引发了我的思考

前端：

package.json中scripts部分

```json
  "scripts": {
    "dev": "vite",
    "build": "run-p type-check \"build-only {@}\" --",
  },
```

启动命令：

~~~shell
npm run dev
~~~

后端

package.json中scripts部分

~~~json
  "scripts": {
    "start": "nodemon ./bin/www"
  },
~~~

启动命令

~~~shell
 npm start
~~~



虽然前后端项目的命令都是写在package.json的scripts中，但是一个需要加run，一个不需要加run这是为什么呢？

![](https://cdn.jsdelivr.net/gh/ilses1/blogs-img/6147fd0e22cb4984b5f2bb09f77f36ee~tplv-k3u1fbpfcp-jj-mark%3A0%3A0%3A0%3A0%3Aq75.image)

# 2.npm run做了什么?

> 要弄清楚为什么一个执行脚本命令npm需要加上run,一个执行脚本命令npm可以不加,我们首先先弄清楚npm run 做了什么

[npm文档： npm-run-script 详解](https://docs.npmjs.com/cli/v10/commands/npm-run-script)

## 2.1终端输入命令

首先我们在终端输入比如`npm run <command> [-- <args>]`的命令

```
npm run dev 
```

## 2.2 到package.json的scripts中寻找命令

在终端输入了如下面命令后

```shell
npm run <command>
```

npm得到command命令,然后到package.json的"scripts"中寻找相同的命令,然后运行命令,

如果没有找到则会提示没有该命令,并告诉我们输入`npm run`可以列出所有scripts列表

![image-20231110154805001](111/image-20231110154805001.png)

输入`npm run`可以显示所有可用scripts:

![image-20231110154921589](111/image-20231110154921589.png)

## 2.3执行命令

例如我执行了启动命令：

~~~shell
npm run dev
~~~

package.json中scripts部分如下:

```json
  "scripts": {
    "dev": "vite"
  }
```

npm在找到dev命令之后,需要执行vite命令,

- npm首先会到当前项目的`node_modules\.bin`目录下找vite命令
- 找不到就到系统环境变量中找vite 

找到后执行vite命令启动项目



# 3.为什么有的npm scripts命令不用run

[npm文档： npm-run-script 详解](https://docs.npmjs.com/cli/v10/commands/npm-run-script)

查看npm 文档可知`test、start、restart 和 stop`四个scripts命令是可以直接调用的命令,(使用run也可以),原因是他们是生命周期(测试,启动,重启)和直接运行的脚本.

> 原文如下

```
run[-script] is used by the test, start, restart, and stop commands, but can be called directly, as well. When the scripts in the package are printed out, they're separated into lifecycle (test, start, restart) and directly-run scripts.
```

也就是说其实`npm start`和`npm run start `效果是一样的,如下都能启动node项目:

![image-20231110165818020](111/image-20231110165818020.png)

![image-20231110165844500](111/image-20231110165844500.png)

当然`test、restart 和 stop`也是这样,

当然前提是先要在package.json中scripts部分中定义command.例子如下

```json
  "scripts": {
    "start": "nodemon ./bin/www",
    "test": "echo \"Error: no test specified\" && exit 1",
    "restart": "pm2 restart server",
    "stop": "pm2 stop server"
  },
```

# 4.总结

>npm run 执行步骤如下:

1.终端输入命令`npm run <command>`

2.npm







