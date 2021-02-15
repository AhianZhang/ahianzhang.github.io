---
title: "项目总结 第二篇"
date: "2018-01-30 22:41:20"
tags: ["项目总结"]
categories: ["总结"]
---
button 的 onclick 时间绑定 submitData 利用 AJAX  进行数据传输

```javascript
    function submitData(){
        var url="<%=request.getContextPath()%>/updateGoodsCat.do";
        var catId=$("#catId").val();
        var name=$("#name").val();
        var title=$("#title").val();
        var pid=$("#pid").val();
        var comments=$("#comments").val();
        var params={
            "catId":catId,
            "name":name,
            "title":title,
            "pid":pid,
            "comments":comments,};
        $.post(url,params,function(result){
           // alert(result);
            //window.location.reload();
        });
    }
```

当 时间为 AJAXSubmit 时 

```javascript
    $("#radio1").click(function () {
        $("#isShow").val($("#radio1").val());
    });
    $("#radio2").click(function () {
        $("#isShow").val($("#radio2").val());
    })
    function submitData(){
        var title1=$("#title1").val();
        var title2=$("#title2").val();
        var cateId=$("#cateId").combotree('getValue');
        var author=$("#author").val();
        console.log(cateId);
        var content = UE.getEditor('container').getContent();
        var stemFrom=$("#stemFrom").val();
        var amount=$("#amount").val();
        var isShow = $("#isShow").val();
        var params={
            "title1":title1,
            "title2":title2,
            "cateId":cateId,
            "author":author,
            "content":content,
            "stemFrom":stemFrom,
            "amount":amount,
            "isShow":isShow
        };
        $("#form1").ajaxSubmit({
            type:"post",
            url:"<%=request.getContextPath()%>/cpyAddArticle.do",
            data:{},
            success:function(msg){
                alert(msg);
                location.reload();
            }
        });
    }
```
