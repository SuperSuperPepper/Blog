# Blog
博客仓库

## 添加图片：

添加图片的话，主要md文件名必须为index.md，图片文件要与其在相同目录。

```text
+---测试文章
|       cover.jpg
|       Design-V1.jpg
|       Design-V2.jpg
|       index.md
```

测试方法：

```
hugo server -D // -D means destination
```

部署：

```
hugo -D
```

## 注意：

标题段落会直接影响hugo是否渲染该篇文章，所以格式不要输入错误。时间错误，冒号错误都会出错。时间写到年月日即可。冒号后面需要跟一个空格。

draft 如果是true的话，不开启特定设置是不会被渲染的，所以不要使用draft =true。
