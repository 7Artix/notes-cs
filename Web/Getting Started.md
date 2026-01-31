# JavaScript 和 Java

JavaScript和Java其实没有直接的关系, 最初的JavaScript甚至不是这个名字. 

当时Java语言正火, 被认为是编程界的未来, 为了蹭热度, 网景公司和拥有Java版权的Sun公司达成协议, 将LiveScript改名为JavaScript.


# Hello, World!

```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
    <title>My First Page</title>
</head>
<body>
    <h1>Page Test</h1>

    <script>
        document.write("<p>Hello, World! from document.write</p>");

        console.log("Hello, World! from console");

        alert("Hello, World! from a pop-up");
    </script>
</body>
</html>
```

`JavaScript` 原生运行在浏览器中. 使用浏览器运行上述代码.

`JavaScript` 也不一定嵌套在 `HTML` 中, 可以新建一个 `script.js`

```js
console.log("独立文件中的终端打印");
alert("独立文件的弹出窗口");
```

并在 `index.html` 中引用:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
</head>
<body>
    <h1>My Page</h1>
    <script src="script.js"></script>
</body>
</html>
```

同理, `CSS` 文件也可以和 `JavaScript` 一样嵌套, 或独立于 `HTML` 文件.

