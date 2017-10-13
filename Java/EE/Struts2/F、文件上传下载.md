[TOC]

# 文件上传

## 1. 引入相应的jar包

> commons-fileupload-1.2.1.jar   
> commons-io-1.3.2.jar

## 2.修改Form 表单属性

把form的`enctype`设置为`multipart/form-data`

```html
<form action="<%=basePath%>upload/upload.action" method="post" name="form" enctype="multipart/form-data">  
    文件1:<input type="file" name="upload"/><br/>  
    <input type="submit" value="上传" />  
</form>  
```

## 3.在action类中添加属性

```java
package com.theliang.fileupload;
import java.io.File;
import org.apache.commons.io.FileUtils;
import org.apache.struts2.ServletActionContext;
import com.opensymphony.xwork2.ActionSupport;

public class FileUpload extends ActionSupport {

	//得到上传的文件，对应表单：<input type="file" name="file1">
	private File file1;
	//得到上传文件的名称
	private String file1FileName;
	//文件的类型（MIME）
	private String file1ContentType;
	public void setFile1(File file1) {
		this.file1 = file1;
	}
	public void setFile1FileName(String file1FileName) {
		this.file1FileName = file1FileName;
	}
	public void setFile1ContentType(String file1ContentType) {
		this.file1ContentType = file1ContentType;
	}
	
	public String execute() throws Exception {
		/*********拿到上传的文件，进行处理**********/
		//把文件上传到upload目录
		//获取上传的目录路径
		String path=ServletActionContext.getServletContext().getRealPath("upload");
		//创建目标文件对象
		File destFile=new File(path,file1FileName);
		//把上传的文件拷贝到目标文件夹中
		FileUtils.copyFile(file1,destFile);
		return SUCCESS;
	}
}
```

> 注意，如果在上传的过程中文件的大小超过了struts2默认的文件大小的话，就会上传失败，这时候，可以根据具体的情况设置`struts.multipart.maxSize`的值来满足上传的需求。

# 多文件上传

## 1. 引入相应的jar包

> commons-fileupload-1.2.1.jar   
> commons-io-1.3.2.jar

## 2.修改Form 表单属性

把form的`enctype`设置为`multipart/form-data`

```html
<form action="<%=basePath%>upload/upload" method="post" name="form" enctype="multipart/form-data">  
    文件1:<input type="file" name="upload"/><br/>  
    文件2:<input type="file" name="upload"/><br/>  
    文件3:<input type="file" name="upload"/><br/>  
    <input type="submit" value="上传" />  
</form> 
```

## 3.在action类中添加的属性都是以数组的形式

```java
package com.theliang.fileupload;
import java.io.File;
import org.apache.commons.io.FileUtils;
import org.apache.struts2.ServletActionContext;
import com.opensymphony.xwork2.ActionSupport;

public class HelloWorldAction {  
    private File[] upload;//得到上传的文件  
    private String[] uploadContentType;//得到上传文件的扩展名  
    private String[] uploadFileName;//得到上传文件的名称  
              
    public File[] getUpload() {  
        return upload;  
    }  
    public void setUpload(File[] upload) {  
        this.upload = upload;  
    }  
    public String[] getUploadContentType() {  
        return uploadContentType;  
    }  
    public void setUploadContentType(String[] uploadContentType) {  
        this.uploadContentType = uploadContentType;  
    }  
    public String[] getUploadFileName() {  
        return uploadFileName;  
    }  
    public void setUploadFileName(String[] uploadFileName) {  
        this.uploadFileName = uploadFileName;  
    }  
      
    public String upload() throws IOException {  
        String realpath = ServletActionContext.getServletContext().getRealPath("/upload");  
        if(upload != null) {  
            for(int i=0; i<upload.length; i++) {  
                File savefile = new File(realpath,uploadFileName[i]);  
                if(!savefile.getParentFile().exists()) {  
                    savefile.getParentFile().mkdirs();  
                }  
                FileUtils.copyFile(upload[i], savefile);  
            }  
            ActionContext.getContext().put("msg", "文件上传成功！");  
        }
        return "success";
    }
}
```

# 文件下载



```java
package com.theliang.fileupload;

import java.io.File;
import java.io.InputStream;
import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.util.Map;

import org.apache.struts2.ServletActionContext;

import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionSupport;

/**
 * 文件下载
 * 1. 显示所有要下载文件的列表
 * 2. 文件下载
 * @author Administrator
 *
 */
public class FileDown extends ActionSupport{

	//方法代码请看下面
	
}
```

## 显示所有下载文件

```java
/*************1.显示所有要下载文件的列表 ***************/
public String list(){
	//得到upload目录路径 
	String path=ServletActionContext.getServletContext().getRealPath("/upload");
	//目录对象
	File file=new File(path);
	//得到所有要下载文件的文件名
	String[] fileNames=file.list();
	//保存
	ActionContext ac=ActionContext.getContext();
	//得到代表request的map(第二种方式)
	Map<String, Object> request=(Map<String, Object>)ac.get("request");
	request.put("fileNames",fileNames);
	return "list";
}
```


## 下载文件

```java
/*************2.文件下载 ***************/
//1.获取请求下载的文件名
private String fileName;
public void setFileName(String fileName){
	System.out.println("原来fileName:"+fileName);
//处理传入的参数中的编码问题
//		try {
//			fileName=new String(fileName.getBytes("ISO8859-1"),"utf-8");
//		} catch (UnsupportedEncodingException e) {
//			e.printStackTrace();
//			throw new RuntimeException(e);
//		}
	//把处理好的文件名赋值
	System.out.println("fileName:"+fileName);
	this.fileName = fileName;
}

//2.下载提交的业务方法（在struts.xml中配置返回stream）
public String down(){
	return "download";
}

//3.返回文件流的方法
public InputStream getAttrInputStream() {
	return ServletActionContext.getServletContext().getResourceAsStream("/upload/"+fileName);
}

//4.下载显示的文件名(浏览器显示的文件名)
public String getDownFileName(){
	//需要进行中文编码
	try {
		fileName=URLEncoder.encode(fileName,"utf-8");
	} catch (Exception e) {
		e.printStackTrace();
		throw new RuntimeException(e);
	}
	System.out.println("DownFileName:"+fileName);
	return fileName;
}
```

## 配置struts.xml文件下载

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
          "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
          "http://struts.apache.org/dtds/struts-2.0.dtd">
<struts>
	<package name="upload" namespace="/" extends="struts-default">

<!-- 配置文件下载的action -->
		<action name="down_*" class="com.theliang.fileupload.FileDown" method="{1}">
			<result name="{1}">/f/{1}.jsp</result>
			<result name="download" type="stream">

				<!-- 自动获取参数值 -->
				<param name="contentType">application/cotet-stream</param>
				<param name="inputName">attrInputStream</param>
				<param name="contentDisposition">attachment;filename=${downFileName}</param>
				<param name="bufferSize">1024/</param>
			</result>
		</action>
	</package>
</struts>
```

