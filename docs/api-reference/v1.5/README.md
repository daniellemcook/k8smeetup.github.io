
## 概要
从 markdown 静态编译 html，包括将代码片段分组到任意选项卡。



## 代码示例
\> bdocs-tab:kubectl 部署运行三个 nginx 实例（最多回滚 10个版本）



bdocs-tab:tab 将在渲染过程中被剥离，和 CSS 一起用于显示或隐藏首选选项卡,由于块引用没有特定的语法高亮，因此由 kubectl 来指定所需的选项卡。

```
bdocs-tab:kubectl_yaml
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
```



bdocs-tab:tab_lang 用于指示这些代码段属于哪个选项卡。tab 部分的字符串指示了选项卡,而语言被放到了下划线之外。在渲染过程中，语言将正确高亮，就好像 bdoc 标记被省略一样。



## 动机

这是一个扩展markdown 文档的项目，使用目录和代码片段窗格在 html 中呈现它们。这种类型的大多数项目都严重依赖于使用 JavaScript/jQuery 进行前端解析。该项目使用NodeJS、Marked 和 highlight.js 来输出语法高亮显示的代码块。



在 blockquotes 和代码块上使用特殊的标记，这些块就可以根据它的相关性来放置。例如：多语言代码块应该分组到在任意选项卡下。



克隆代码库，然后将文档添加到文档目录中。按照所需的顺序，修改 manifest.json 以包含文档文件名。docs 字段是一个以文件名为 key 的对象数组。



作为一个 NodeJS 程序，需要对 node 进行有效的安装。node 安装完成后，还需要验证是否可以从命令行运行。

```
node --version
```



接下来，在项目的根目录下面使用 npm 安装依赖项。

```
npm install
```


依赖项安装完成后，运行

```
node brodoc.js
```



这将生成 index.html 文件，可以在浏览器中打开它。



可以通过如下命令，运行项目根目录下所包含的静态节点服务器

```
npm start
```



## 许可证

Apache License Version 2.0

## FAQ

问：为什么它叫做 brodocs ？

答：这个项目源于与我兄弟的合作，按照他的要求制作一个合适的文档应用程序。对于我们两个真正的兄弟来使用的话，这倒是一个有趣的名字。
