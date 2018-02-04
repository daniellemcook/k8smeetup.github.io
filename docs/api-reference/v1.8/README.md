## 概要


静态编译 html 文件，这些 html 文件来源于需要将代码片段分组为任意选项卡进行处理的 markdown 文件。


## 代码示例


\> bdocs-tab:kubectl 部署运行三个 nginx 实例（最大修改次数设置为 10）


在渲染过程中，bdocs-tab:tab 将被剥离，并与 CSS 一起用于显示或隐藏首选选项卡。因为 blockquotes 没有特定的语法高亮，所以 kubectl 指示所需的选项。

\`\`\`bdocs-tab:kubectl_yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: deployment-example
spec:
  replicas: 3
  revisionHistoryLimit: 10
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.10
\`\`\`


bdocs-tab:tab_lang 用于指示这些代码片段属于哪个选项卡。字符串的 tab 部分指示选项卡，而语言则被推送到下划线之外。在渲染过程中，语言能够被正常高亮显示，就像省略了 bdoc 令牌一样。


## 动机


这是一个扩展标记文档的项目，并在 html 中使用目录和代码片段窗格呈现它们。这种类型的大多数项目都严重依赖于使用 JavaScript/jQuery 进行前端解析。该项目使用NodeJS、Marked 和 highlight.js 来输出语法高亮显示的代码块。


在 blockquote 和代码块上使用特殊的标记，块就可以根据它的相关性来放置。例如：多语言代码块应该在任意选项卡下分组。


## 安装


克隆代码库，然后将文档添加到文档目录中。按照所需的顺序，修改 manifest.json 以包含文档文件名。docs 字段是一个以文件名为 key 的对象数组。


作为一个 NodeJS 程序，需要 node 的有效安装。node 安装完成后，验证其是否可以从命令行运行。
```
node --version
```

接下来，在项目的根目录下面使用 npm 安装依赖项。
```
npm install
```


待依赖安装完成后，运行。
```
node brodoc.js
```


这将生成 index.html 文件，可以在浏览器或服务中打开它。


可以从项目根目录下运行所包含的节点静态服务器。
```
npm start
```


## 许可证

Apache License Version 2.0


## 常见问题解答


问：为什么它叫做 brodocs?
答：这个项目源于与我哥哥的合作，是为了给他制作一个合适的文档应用程序。我们两个真正的兄弟来使用的话，这是一个有趣的名字。