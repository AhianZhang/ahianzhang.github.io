---
title: "项目总结 第一篇"
date: "2018-01-30 21:58:49"
tags: ["项目总结"]
categories: ["总结"]
---
在公司做一个商业的网站，里面有很多值得学习和保存的内容。

这次是在未提交表单时进行图片回显的JavaScript代码,很棒的效果。 
以下仅显示用法
```html
 <tr>
                <th>封面图</th><td>
                   <input class="easyui-filebox" style="width:300px;" data-options='onChange:change_photo' id="file" name="file" buttonText='选择图片' value="${goods.thumb }"/>
      
                <!-- <input class="ipt" style="width:400px" name="thumb"  id="thumb" > -->
                </td>
            </tr>
            
            <tr>
            <th>图片预览</th><td><div id="Imgdiv">
              <img id="Img" width="200px" height="200px" src="<%=request.getContextPath()%>${goods.thumb}"/>
               </div></td>
            </tr>
```



```javascript
<script type="text/javascript">
function change_photo(){
    PreviewImage($("input[name='file']")[0], 'Img', 'Imgdiv');
}

function PreviewImage(fileObj,imgPreviewId,divPreviewId){    
    var allowExtention=".jpg,.bmp,.gif,.png";//允许上传文件的后缀名document.getElementById("hfAllowPicSuffix").value;    
    var extention=fileObj.value.substring(fileObj.value.lastIndexOf(".")+1).toLowerCase();                
    var browserVersion= window.navigator.userAgent.toUpperCase();    
    if(allowExtention.indexOf(extention)>-1){     
        if(fileObj.files){//HTML5实现预览，兼容chrome、火狐7+等    
            if(window.FileReader){    
                var reader = new FileReader();     
                reader.onload = function(e){    
                    document.getElementById(imgPreviewId).setAttribute("src",e.target.result);    
                }      
                reader.readAsDataURL(fileObj.files[0]);    
            }else if(browserVersion.indexOf("SAFARI")>-1){    
                alert("不支持Safari6.0以下浏览器的图片预览!");    
            }    
        }else if (browserVersion.indexOf("MSIE")>-1){    
            if(browserVersion.indexOf("MSIE 6")>-1){//ie6    
                document.getElementById(imgPreviewId).setAttribute("src",fileObj.value);    
            }else{//ie[7-9]    
                fileObj.select();    
                if(browserVersion.indexOf("MSIE 9")>-1)    
                    fileObj.blur();//不加上document.selection.createRange().text在ie9会拒绝访问    
                var newPreview =document.getElementById(divPreviewId+"New");    
                if(newPreview==null){    
                    newPreview =document.createElement("div");    
                    newPreview.setAttribute("id",divPreviewId+"New");    
                    newPreview.style.width = document.getElementById(imgPreviewId).width+"px";    
                    newPreview.style.height = document.getElementById(imgPreviewId).height+"px";    
                    newPreview.style.border="solid 1px #d2e2e2";    
                }    
                newPreview.style.filter="progid:DXImageTransform.Microsoft.AlphaImageLoader(sizingMethod='scale',src='" + document.selection.createRange().text + "')";                                
                var tempDivPreview=document.getElementById(divPreviewId);    
                tempDivPreview.parentNode.insertBefore(newPreview,tempDivPreview);    
                tempDivPreview.style.display="none";                        
            }    
        }else if(browserVersion.indexOf("FIREFOX")>-1){//firefox    
            var firefoxVersion= parseFloat(browserVersion.toLowerCase().match(/firefox\/([\d.]+)/)[1]);    
            if(firefoxVersion<7){//firefox7以下版本    
                document.getElementById(imgPreviewId).setAttribute("src",fileObj.files[0].getAsDataURL());    
            }else{//firefox7.0+                        
                document.getElementById(imgPreviewId).setAttribute("src",window.URL.createObjectURL(fileObj.files[0]));    
            }    
        }else{    
            document.getElementById(imgPreviewId).setAttribute("src",fileObj.value);    
        }             
    }else{    
        alert("仅支持"+allowExtention+"为后缀名的文件!");    
        fileObj.value="";//清空选中文件    
        if(browserVersion.indexOf("MSIE")>-1){                            
            fileObj.select();    
            document.selection.clear();    
        }                    
        fileObj.outerHTML=fileObj.outerHTML;    
    }    
}  

</script>
```
