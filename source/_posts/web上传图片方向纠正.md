---
layout: post
title: web上传图片方向纠正
date: 2018-09-10
categories: iOS开发 web
tags: [iOS, WebView]
---

最近做的应用中有一个WKWebView加载了一个可以上传图片的网页，web端用`<input type='file'>`标签上传文件。
随后发现在PC端用`<img>`展示的时候，图片发生了偏转。

具体大概是这个样子的，客户吐槽：多年的颈椎病都被治好了。
![方向错误的图片](https://snowyblog.oss-cn-shenzhen.aliyuncs.com/webImageOrientation6.png)

<!-- more -->

因为只有通过iOS客户端上传的图片才会偏转，所以问了负责Android客户端的同学，原来Android的webView在文件上传前有回调，在回调里面对图片方向进行了处理。

于是我翻看了WKWebView头文件中相应代理的回调，并没有这种文件上传的回调，在网上看了一下其他人的解决方案，大致都是通过runtime替换了ViewController的`presentViewController: animated: completion:`和`dismissViewControllerAnimated: completion:`方法。

然后在方法中判断present的viewcontroller是否是`UIDocumentMenuViewController`类型，且代理是`WKFileUploadPanel`或`UIWebFileUploadPanel`，对其进行拦截，由自己接管后续图片的处理和上传，不过上传后，还是需要将图片URL传递给web网页展示，这样的话，Android客户端也需要进行相应的处理，改动就大了，所以放弃在iOS端改动。具体参考链接：[iOS拦截\<input\>标签](https://github.com/frog78/Gigi)

于是就只能在web端改代码了，web端旋转压缩图片的话，主要用的还是canvas。但是获取图片的方向的话，需要用到[exif.js](https://github.com/exif-js/exif-js)；

废话不多说，上代码，首先引入exif.js：

```HTML
<script src="js/exif.js"></script>
```

监听input标签：
```HTML
<input type="file" accept="image/*" class="uploadInput" style="display: none">
<script>
$(".uploadInput").change(function (e) {
    var file = e.target.files[0];
    FixImg(file, function (imgFile) {
        // 上传图片
        var formdata = new FormData();
        formdata.append('file', imgFile);
        $.ajax({
            url: 'upload.do',
            type: 'POST',
            data: formdata,
            processData: false,
            contentType: false,
            success: function (res){
                // 图片上传成功
            }
        })
    })
})
</script>
```
具体压缩旋转函数：
```javaScript
    function FixImg(file, callback) {
        //图片压缩质量
        var quality = 0.65;
        // 定义所需变量
        let Orientation, ctxWidth, ctxHeight, base64; 
        // EXIF 获取图片方向信息
        EXIF.getData(file, function() {
            Orientation = EXIF.getTag(this, 'Orientation');
            
            var fileReader = new FileReader();
            fileReader.readAsDataURL(file);
            fileReader.onload = function (event) {
                
                var res = event.target.result;
                var image = new Image();
                image.src = res;
                image.onload = function () {

                    ctxWidth = this.naturalWidth;
                    ctxHeight = this.naturalHeight;

                    var canvas = document.createElement('canvas');
                    var ctx = canvas.getContext('2d');

                    canvas.width = ctxWidth;
                    canvas.height = ctxHeight;
                    if ([5, 6, 7, 8].includes(Orientation)) {
                        canvas.width = ctxHeight;
                        canvas.height = ctxWidth;
                    }

                    switch (Orientation) {
                        case 2:
                            ctx.transform(-1, 0, 0, 1, ctxWidth, 0);
                            break;
                        case 3:
                            ctx.transform(-1, 0, 0, -1, ctxWidth, ctxHeight;
                            break;
                        case 4:
                            ctx.transform(1, 0, 0, -1, 0, ctxHeight);
                            break;
                        case 5:
                            ctx.transform(0, 1, 1, 0, 0, 0);
                            break;
                        case 6:
                            ctx.transform(0, 1, -1, 0, ctxHeight, 0);
                            break;
                        case 7:
                            ctx.transform(0, -1, -1, 0, ctxHeight, ctxWidth;
                            break;
                        case 8:
                            ctx.transform(0, -1, 1, 0, 0, ctxWidth);
                            break;
                        default:
                            ctx.transform(1, 0, 0, 1, 0, 0);
                    }

                    ctx.drawImage(image, 0, 0, ctxWidth, ctxHeight);

                    canvas.toBlob(function (blob) {
                        // 上传接口只接受file，所以blob转为file再上传
                        let files = new window.File([blob], file.name, {type: file.type})
                        callback(files);
                    }, 'multipart/form-data', quality)

                }
            }

        });
    };

```



