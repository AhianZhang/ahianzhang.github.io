---
title: "editor.md 富文本编辑器的使用"
date: "2018-01-30 22:08:08"
tags: ["项目总结"]
categories: ["总结"]
---
editormd 是一款国产的软件，虽然很久没更新了，但其所实现的功能已满足日常使用，下面主要讲使用时需要注意的点。
<!-- more -->
在 Body 中写两个文本域便于保存,注意 class
```html
<div id="articlemd" style="margin-top: 10px">
                <textarea class="editormd-markdown-textarea" name="content"></textarea>
                <!-- 第二个隐藏文本域，用来构造生成的HTML代码，方便表单POST提交，这里的name可以任意取，后台接受时以这个name键为准 -->
                <textarea class="editormd-html-textarea" name="editorhtml"></textarea>
            </div>
```

在取编辑器中的几行内容是用到的是下面的 JavaScript 方法
```javascript
var summary = testEditor.setSelection({line: 0, ch: 0}, {line: 4, ch: 100});
summary.getSelection();
```

取得编辑器中所有的内容，以便保存到数据库中，注意数据库最好保存的是 Markdown 格式的。

```javascript
var content = testEditor.getMarkdown();

```

在前台显示数据时，调用解析器如下，注意ID值要和 JavaScript 代码对应
```javascript
  <div class="blog_content" id="content">

            <textarea style="display:none;" name="editormd-markdown-doc"> ${blog.content}</textarea>
 </div>
```

```javascript

    $(document).ready(function () {
        var wordsView;
        wordsView = editormd.markdownToHTML("content", {
            htmlDecode: "style,script,iframe",  // you can filter tags decode
            emoji: true,
            taskList: true,
            tex: true,  // 默认不解析
            flowChart: true,  // 默认不解析
            sequenceDiagram: true,  // 默认不解析
        });

    })
```

 目前就这么多，想起来再补充。