# 基于fetch封装的request库

# request()
request链式处理
```js
request.request('/file/root',{
	params:{},//参数列表
	method:'GET',// 请求方式,默认是GET
	headers:{//请求头
		'Content-Type': 'application/x-www-form-urlencoded',//默认是application/x-www-form-urlencoded
	},
})
.then(res=>res.json()) //转为json
.then(data=>data.data) // 获取响应数据中的 data
...then(data=>//todo)
.then(data=>{
	console.log(data) // 数据体
})
```

## request的GET请求
```js
request.request('/file/path',{
	params:{},
	method:'get'
})
.then(res=>res.json())
.then(data=>data.data)
.then(data=>{
	console.log(data)
})
```
## request的POST请求
```js
request.request('/file/path',{
	params:{},
	method:'POST'
})
.then(res=>res.json())
.then(data=>data.data)
.then(data=>{
	console.log(data)
})
```

# get请求
```js
request.get('/file/root')
.then(res=>res.json())
.then(data=>data.data)
.then(data=>console.log(data))
```
```js
// 带参数
request.get('/file/path',{path:'/RDpcbWFpblxxaW5qaWF3YW5nXHByb2plY3RcZmlsZS1tYW5hZ2Vy/data'})
.then(res=>res.json())
.then(data=>data.data)
.then(data=>console.log(data))
```

# post请求
```js
request.post('/file/del/file',{path:'/RDpcbWFpblxxaW5qaWF3YW5nXHByb2plY3RcZmlsZS1tYW5hZ2Vy/data'})
.then(res=>res.json())
.then(data=>data.data)
.then(data=>console.log(data))
```

# postJson 
```js
request.postJson('/file/del/file',{path:'/RDpcbWFpblxxaW5qaWF3YW5nXHByb2plY3RcZmlsZS1tYW5hZ2Vy/data'})
.then(res=>res.json())
.then(data=>data.data)
.then(data=>console.log(data))
```

# upload 文件上传

```html
<input type ="file" id="upload" onclick="upload()"> 
```
```js
function upload(){
	let uploadInput = document.getElementById('upload')
	uploadInput.addEventListener('change',function(e){
		let file = e.target.files[0]
		//1.构建一个表单
		var fd=new FormData();
		//2.添加文件值
		fd.append('file',file);
		fd.append('path','/RDpcbWFpblxxaW5qaWF3YW5nXHByb2plY3RcZmlsZS1tYW5hZ2Vy/data');
		//3.上传
        request.upload('/upload/file',fd,{
			before:function () {},//上传之前回调
			ready:function(){},//开始上传时回调
			progress:function (percent) {//上传进度回调
				console.log(percent)
			},
			after:function () {},//上传后回调
			success:function () {},//上传成功后回调
			error:function () {},//上传出错回调
          }
		)
	},false); 
}
```
# uploadFile 文件上传
函数原型
```js
/**
	* @param url
	* @param file 文件对象 如果是一个数组则取第一个
	* @param callback 回调函数
	* @param name 文件对象name名称
	*/
	uploadFile:function(url,file,callback,name='file')
```
```js
function upload(){
	let uploadInput = document.getElementById('upload')
uploadInput.addEventListener('change',function(e){	
		let file = e.target.files
		
        request.uploadFile('/upload/file',file[0],{
			before:function () {},//上传之前回调
			ready:function(){},//开始上传时回调
			progress:function (percent) {//上传进度回调
				console.log(percent)
			},
			after:function () {},//上传后回调
			success:function () {},//上传成功后回调
			error:function () {},//上传出错回调
          },"files"
		)
	},false); 
}
```

## 文件上传 Java后端实现
```java
/**
     * 上传文件
     * @return
     */
    @RequestMapping(value = "/file",method = RequestMethod.POST )
    public R imgUpload(@RequestParam(value = "file",required = false) MultipartFile file, String path) {

        if(file==null){
            return R.isFail();
        }
        InputStream inputStream = null;
        try {
            inputStream = file.getInputStream();
        } catch (IOException e) {
            return R.isFail();
        }
        String filename = file.getOriginalFilename();
		
		// TODO 处理逻辑
        boolean ret = save(inputStream,path);
        return ret ? R.isOK() : R.isFail();
    }
	
	public  void save(InputStream is, String fileName) throws IOException {
        BufferedInputStream in=null;
        BufferedOutputStream out=null;
        in=new BufferedInputStream(is);
        out=new BufferedOutputStream(new FileOutputStream(fileName));
        int len=-1;
        byte[] b=new byte[1024];
        while((len=in.read(b))!=-1){
            out.write(b,0,len);
        }
        in.close();
        out.close();
    }
```
# downloadFile 文件下载
```js
/**
	* @param url
	* @param params {rename:''} 参数列表,可自定义参数,其中rename是重命名文件
	*/
	downloadFile:function(url,params)
```

```js
request.downloadFile('/download/file',{path:'/RDpcbWFpblxxaW5qaWF3YW5nXHByb2plY3RcZmlsZS1tYW5hZ2Vy/data',file:'qjw.zip',rename:'qjw2.zip'})	
```
```java
//java后端实现
@PostMapping("/file")
    public R file(String path, String file, HttpServletResponse response){
        if(Operater.isEmpty(path)){
            return R.isFail().message("path is requird");
        }
        InputStream in = fileService.downloadFile(path,file);
        if(Operater.isNull(in)){
            return R.isFail().message("file is not found.");
        }
        download(in,response,file);
        return R.isOK();
    }
	
	 private void download(InputStream in,HttpServletResponse response,String file){
        try {

            // 设置响应头，控制浏览器下载该文件
            response.setHeader("content-disposition", "attachment;filename=" + URLEncoder.encode(file, "UTF-8"));
            // 读取要下载的文件，保存到文件输入流
            response.setHeader("Access-Control-Expose-Headers", "Content-Disposition");
            // 创建输出流
            OutputStream out = null;
            out = response.getOutputStream();
            // 创建缓冲区
            byte buffer[] = new byte[1024];
            int len = 0;
            // 循环将输入流中的内容读取到缓冲区当中
            while ((len = in.read(buffer)) > 0) {
                // 输出缓冲区的内容到浏览器，实现文件下载
                out.write(buffer, 0, len);
            }
            // 关闭文件输入流
            in.close();
            // 关闭输出流
            out.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```