# upload
H5 fileReader对象实现图片上传（支持一次上传多张图片），并用canvas进行前端压缩<br>
首先页面布局的代码是这样的<br>
将type="file"的input绝对定位到上传图片的图标上，并且将opacity设置为0，即可实现样式效果，accept="image/*" 代表会打开图库或者照相机，multiple代表可支持一次上传多张图片，具体代码如下，详细见源代码upload/upload.html，upload/css/my.css
```javascript
<!-- 上传图片 start -->
<div class="car-sale">
    <p>上传图片（首张图片为车辆正前方45°照片，建议横屏拍摄）</p>
    <div class="sale-upload border-box clearfix" id="upload-img">
        <div class="upload-add border-box auto-height">
            <img src="images/upload-add.jpg">
            <input type="file" name="file_upload" id="file_upload" accept="image/*" multiple>
        </div>
        <!-- <div class="upload-img border-box auto-height">
            <img src="images/car-5.jpg">
            <a href="javascript:;" class="delete-img"></a>
        </div> -->
    </div>
</div>
<!-- 上传图片 end -->
```
<br>
<br>上传图片的js代码实现部分请见upload/js/upload.js，主要借助了h5的fileReader对象以及canvas进行前端压缩后再将参数转化为base64位编码格式传给后台，避免图片过大造成服务器的压力，具体代码详解请见如下代码注释部分。<br>
```javascript
var picList = [];  //上传图片地址
$('#file_upload').change(function(){  
	if (!this.files.length) return;
	var files = Array.prototype.slice.call(this.files);
	if (files.length > 9) {
			Uton.tip("最多同时只可上传9张图片");
	      return;
	    }
	files.forEach(function(file, i) {
	    if (!/\/(?:jpeg|png|gif)/i.test(file.type)) return;  
            //$.notify.show('图片上传中...', function(){});
	    
        var fileReader = new FileReader();  
        fileReader.readAsDataURL(file);  
        图片上传部分代码       
        } 
	})
});
```
<br>图片上传部分代码如下：
```javascript
fileReader.onload = function(event){  
	var result = event.target.result;   //返回的dataURL  
	var image = new Image();  
	image.src = result;  
	image.onload = function(){  //创建一个image对象，给canvas绘制使用  
	var cvs = document.createElement('canvas');  
	var scale = 1;    
	if(this.width > 1000 || this.height > 1000){  //1000只是示例，可以根据具体的要求去设定    
	    if(this.width > this.height){    
		scale = 1000 / this.width;  
	    }else{    
		scale = 1000 / this.height;    
	    }    
	}  
	cvs.width = this.width*scale;    
	cvs.height = this.height*scale;     //计算等比缩小后图片宽高  
	var ctx = cvs.getContext('2d');    
	ctx.drawImage(this, 0, 0, cvs.width, cvs.height);     
		      var newImageData = cvs.toDataURL(file.type, 0.8);   //重新生成图片，<span style="font-family: Arial, Helvetica, sans-serif;">fileType为用户选择的图片类型</span>  
	var sendData = newImageData.replace("data:"+file.type+";base64,",'');  
	var urlList = [];
	urlList.push(sendData);                
	$.ajax({ 
	    url : '/tool/upload',   //接口地址
	    type : 'get', 
	    data : {
		"file": urlList
	    },
	    traditional: true, 
	    //cache : false, 
	    //contentType : false, 
	    //processData : false, 
	    dataType : "json", 
	    success : function(data){
		var newImg = '<div class="upload-img border-box auto-height"><img src="' + data.data[0] + '"><a href="javascript:;" class="delete-img"></a></div>'; 
		picList.push(data.data[0]);
		$("#upload-img").append(newImg); 
		var finalWidth = $(".upload-add").width();
		$(".auto-height").height(finalWidth); 
	    } 
	}); 
} 
```
<br>注意：使用时把接口地址改成你的接口地址即可查看效果
