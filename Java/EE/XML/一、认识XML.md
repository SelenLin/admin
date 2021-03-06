
# 什么是XML？

>XML即可扩展标记语言（e<font color="red">**X**</font>tensible <font color="red">**M**</font>arkup <font color="red">**L**</font>anguage）。标记是指计算机所能理解的信息符号，通过此种标记，计算机之间可以处理包含各种信息的文章等。如何定义这些标记，既可以选择国际通用的标记语言，比如HTML，也可以使用象XML这样由相关人士自由决定的标记语言，这就是语言的可扩展性。XML是从SGML中简化修改出来的。它主要用到的有XML、XSL和XPath等。

&emsp;<font size="4">**上**</font>面这段是对XML的一个基本定义，一个被广泛接受的说明。简单说，XML就是一种数据的描述语言，虽然它是语言，但是通常情况下，它并不具备常见语言的基本功能——被计算机识别并运行。只有依靠另一种语言，来解释它，使它达到你想要的效果或被计算机所接受。

&emsp;<font size="4">**假**</font>如你是刚接触XML的新手，那么可能并无法从定义上是了解XML是什么。也许，你可以换个角度来认识XML是什么；从应用面来认识XML，从XML可以做些什么来认识它，这应该能比那更空洞的定义对你更有帮助。

&emsp;<font size="4">**XML**</font>应用面主要分为两种类型，<font color="red">文档型</font>和<font color="red">数据型</font>。下面介绍一下几种常见的XML应用：<br/><br/>
&emsp;<font size='4'>**1、**</font>自定义XML+XSLT=>HTML，最常见的文档型应用之一。XML存放整个文档的XML数据，然后XSLT将XML转换、解析，结合XSLT中的HTML标签，最终成为HTML，显示在浏览器上。典型的例子就是CSDN上的帖子。

&emsp;<font size='4'>**2、**</font>XML作为微型数据库，这是最常见的数据型应用之一。我们利用相关的XML API（MSXML DOM、JAVA DOM等）对XML进行存取和查询。留言板的实现中，就经常可以看到用XML作为数据库。同时，这里要告诉一些新人，数据库和数据库系统，这两个概念是不同的。这里顺便提一下XML对数据库系统的影响。在新版本的传统数据库系统中，XML成为了一种数据类型。和“传统”相对的就是一种新形态的数据库，完全以XML相关技术为基础的数据库系统。目前比较知名的eXist。

&emsp;<font size='4'>**3、**</font>作为信息传递的载体。为什么说是载体呢？因为这些应用虽然还是以XML为基本形态，但是都已经发展出具有特定意义的格式形态。最典型的就是WEB SERVICE，将数据包装成XML来传递，但是这里的XML已经有了特定的规格，即SOAP。不过这里还不得不说AJAX，AJAX的应用中，相信也有一部分的应用是以自定义XML为数据，不过没有成为工业标准，这里不做详述。

&emsp;<font size='4'>**4、**</font>应用程序的配置信息数据。最典型的就是J2EE配置WEB服务器时用的web.XML。这个应用估计是很容易理解的了。我们只要将需要的数据存入XML，然后在我们的应用程序运行载入，根据不同的数据，做相应的操作。这里其实和应用2，有点类似，所不同的在于，数据库中的数据变化是个常态，而配置信息往往是较为静态，缺少变化的。

&emsp;<font size='4'>**5、**</font>其他一些文档的XML格式。如WORD、EXCEL等。

&emsp;<font size='4'>**6、**</font>保存数据间的映射关系。如Hibernate。


&emsp;<font size="4">**这**</font>几种常见应用中，我们还可以根据其应用广泛程度，分为：自定义XML和特定意义XML。在1和2就是属于自定义XML的范畴；3至6则属于特定意义XML，或者说是XML的延伸。
这里介绍的6种应用，基本涵盖了XML的主要用途。总之，XML是一种抽象的语言，它不如传统的程序语言那么具体。要深入的认识它，应该先从它的应用入手，选择一种你需要的用途，然后再学习如何使用。



# HTML和XML的区别

|     |HTML| XML|
| --- |:------|:------|
|  名称   |   <font color="red" size="4">H</font>yper<font color="red" size="4">T</font>ext <font color="red" size="4">M</font>arkup <font color="red" size="4">L</font>anguae 超文本标记语言      |    e<font color="red" size="4">X</font>tend <font color="red" size="4">M</font>arkup <font color="red" size="4">L</font>anguge可扩展标签语言     |
|  标签   |    标签是w3c组成指定，固定的，约100来个     |   标签由开发者自己制定的(要按照一定的语法定义)      |
|  作用   |   负责网页的结构      |   1.描述带关系的数据（作为软件的配置文件）: 包含与被包含的关系<br/>&emsp;例如：properties文件：[key]-[value]<br/>&emsp;场景：tomcat struts Hibernate spring (三大框架)<br/>2.作为数据的载体(存储数据，小型的“数据库”)  |



# XML的作用

## 1.1 描述带关系的数据（<font color="red">软件的配置文件</font>）
    web服务器（PC）：
        学生管理系统 -> 添加学生功能 -> 添加学生页面 -> name=eric&email=eric@qq.com 

        前提： 网络（IP地址： oracle：255.43.12.54  端口：1521 ）
            java代码：使用ip（255.43.12.54）地址和端口（1521），连接oracle数据库，保存学生数据。

        把ip地址端口配置到xml文件：
              host.xml：
                 <host>
                     <ip>255.43.12.55</ip>
                     <port>1521</port>
                 </host>
 
    数据库服务器（PC）：
        主服务器（255.43.12.54）：Oracle数据库软件（负载）
        副服务器（255.43.12.55）：Oracle数据库软件


##　1.2 数据的载体（小型的“数据库”）
       教师管理系统： 姓名   工龄+1  邮箱
          发教师数据给财务管理系统：
             String teacher =    name=张三&email=zhangsan@qq.com&workage=2  字符串

     （问题： 1.不好解析  ；2.不是规范）
           teacher.xml：
                <teacher>       
                    <name>张三</name>
                    <email>zhangsan@qq.com</email>
                    <workage>2</workage>
                </teacher>
      这是一种规范
 
           财务管理系统：   
                 姓名   工龄+1  邮箱
                 发奖金：   统计奖金。   工龄
                 发邮件功能：邮箱   姓名   金额
 
              方案一： 在财务管理系统中维护了一套教师信息。
                      每年 ： 工龄增加  维护了两个系统的信息。
 
              方案二： 教师信息只在教学管理系统中维护。
