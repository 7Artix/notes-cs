# 端口

在web中, 端口是程序的访问id.

IP地址对应服务器的位置, 而端口号对应的是服务器上跑的具体的一个程序.

一个端口同一时间只能被一个程序监听, 当浏览器请求对应的端口, 操作系统就会将流量链接到正在监听对应端口的进程.

# 数据隔离

前端数据和后端数据是完全隔离的. 用户只能访问 `dist/` 目录下的资源. 在设计阶段, 这些资源属于 `src/` 目录. `dist/` 目录是编译打包后的产物.

其余的目录, 例如 `data/` 和 `server/` 目录只有后端 `Node.js` 才能访问, 而后端需要通过 `api/` 调用.

# URL

URL本身只是字符串, 当可以解析到文件或目录时, 可以作为地址, 请求对应的资源. 当无法访问到对应的目录时, 直接作为参数, 通过vue生成的JavaScript进行解析. 此时, URL只是用于判断该进行如何解析的参数.

# Nginx, Node.js, 浏览器 和 Vue

## Nginx

Nginx负责最顶层的端口监听, 所有请求首先经过Nginx, Nginx负责最初的路由.

Nginx的路由逻辑为:

```nginx
server {
    listen 80;
    server_name 7artix.com;

    location /api {
        proxy_pass http://localhost:3000;
    }

    location / {
        root /var/www/7artix/dist;
        try_files $uri $uri/ /index.html;
    }
}
```


## Node.js

所有 `/api` 请求会一律交由 `Node.js` 处理, 无论是访问静态文件, 还是调用函数, 全部由 `Nginx` 交给 `Node.js` 处理.

以下是 `server/index.js` 的逻辑

```js
import express from 'express';
import cors from 'cors';
import objectRoutes from './routes/objects.js';
import tagRoutes from './routes/tags.js';

const app = express();

// 中间件配置
app.use(cors());
app.use(express.json());

app.use('/api/objects', objectRoutes);
app.use('/api/tags', tagRoutes);

// 3. 静态资源映射
app.use('/api/static', express.static('./data'));

const PORT = 3000;
app.listen(PORT, () => {
    console.log(`Backend Engine running at http://localhost:${PORT}`);
});
```

可以看到, 对应的一些操作类的API, 会路由到对应函数. 对于资源类的访问, 会通过 `express.static()` 方法进行映射, 此处可进行权限检测.


## 浏览器

浏览器可以执行前端JavaScrip和渲染HTML, 不能看到任何和后端程序相关的东西.


## Vue

Vue框架管理前端的行为, 如何渲染, 如何执行


# 运行过程

- 当用户输入网址 `http://7artix.com` 时, 会默认访问对应IP的 `80` 端口. 如果指定了别的端口, 或是使用别的协议, 则以实际目标为准. 
- 用户请求的 `URL` 会首先经过 `Nginx` 的路由逻辑
	- 当请求 `/api` 时, 会直接交给后台的 `Node.js` 处理
	- 当请求 `/` 其余目录时, 首先会设置根目录, 设置能够访问的区域. 然后依次寻找, 请求是否是一个存在的文件, 是否是一个存在的目录, 若存在则返回. 若均不存在, 则交给 `index.html` 处理. 对应URL作为参数.
- 默认请求 `index.html` , 监听 `80` 端口的程序返回给客户端对应的HTML文件, 在客户端执行.
- `index.html` 中会有对应的JavaScript脚本, 例如指定 `main.js` 浏览器会请求对应的脚本执行.
- `main.js` 中设置了对应的 `router` 引用, 在router中有路由逻辑, 对于不同的URL有对应的Vue页面, 即会运行对应的JavaScrip代码, 渲染对应的HTML页面.

