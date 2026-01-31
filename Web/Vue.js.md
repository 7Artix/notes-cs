# 关系

Vue.js是JavaScript库.

当创建项目时, `npm` 会将Vue库下载到项目文件夹 ( `node_modules` 目录).

Vue是一个模板, 类似的还有React模板.


# Vite

Vite是构建工具, 负责提供开发环境, 编译代码, 打包资源.

Vite不强绑定Vue.js, 当运行 `npm create vite` 时, 本质上是下载了 `create-vite` 程序, 程序内置了多种模板, 例如Vue, React等.


# 使用 Vite 创建项目

运行如下命令:

```shell
artix@Organ ~/Projects> npm create vite@latest 7artix.com

> npx
> "create-vite" 7artix.com

│
◇  Select a framework:
│  Vue
│
◇  Select a variant:
│  JavaScript
│
◇  Use rolldown-vite (Experimental)?:
│  No
│
◇  Install with npm and start now?
│  Yes
│
◇  Scaffolding project in /Users/artix/Projects/7artix.com...
│
◇  Installing dependencies with npm...

added 34 packages, and audited 35 packages in 4s

6 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
│
◇  Starting dev server...

> 7artix.com@0.0.0 dev
> vite


  VITE v7.3.1  ready in 1580 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

进入项目目录后, 可以输入:

```shell
npm run dev
```

即可在浏览器看到实时预览.

使用 `npm install <包名>` 安装依赖包, 安装后的包会记录到 `package.json` 中. 便于后续移植.


# 网页标题和网页图标

是网站的元数据Metadata, 在Vue中, 分为静态设置(全局不变)和动态设置(切换页面时改变).

## 静态设置 (全局默认值)

在Vue

```html

```