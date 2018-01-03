**马哥的淘宝店:https://shop592330910.taobao.com/**


##文件上传

1.进行文件上传时, 表单需要做的准备:

1). 请求方式为 POST: `<form action="uploadServlet" method="post" ... >`
2). 使用 file 的表单域: `<input type="file" name="file"/>`
3). 使用 multipart/form-data 的请求编码方式: `<form action="uploadServlet" method="post" enctype="multipart/form-data">`
```
<form action="uploadServlet" method="post" enctype="multipart/form-data">

    File: <input type="file" name="file"/>
    <input type="submit" value="Submit"/>

</form>
```
4). 关于 enctype:

    > application/x-www-form-urlencoded：表单 enctype 属性的默认值。这种编码方案使用有限的字符集，当使用了非字母和数字时，
    必须用”%HH”代替(H 代表十六进制数字)。对于大容量的二进制数据或包含非 ASCII 字符的文本来说，这种编码不能满足要求。
    
    > multipart/form-data：form 设定了enctype=“multipart/form-data”属性后，表示表单以二进制传输数据 

2.服务端:

1). 不能再使用 request.getParameter() 等方式获取请求信息. 获取不到, 因为请求的编码方式已经改为 multipart/form-data, 以
二进制的方式来提交请求信息. 

2). 可以使用输入流的方式来获取. 但不建议这样做.

3). 具体使用 commons-fileupload 组件来完成文件的上传操作. 

3.搭建环境: 加入 
commons-fileupload-1.2.1.jar
commons-io-2.0.jar

II. 基本思想: 

    > commons-fileupload 可以解析请求, 得到一个 FileItem 对象组成的 List
    > commons-fileupload 把所有的请求信息都解析为 FileItem 对象, 无论是一个一般的文本域还是一个文件域. 
    > 可以调用 FileItem 的 isFormField() 方法来判断是一个 表单域 或不是表单域(则是一个文件域)
    > 再来进一步获取信息
    
```
    if (item.isFormField()) {
        String name = item.getFieldName();
        String value = item.getString();
    
    } 
    
    if (!item.isFormField()) {
        String fieldName = item.getFieldName();
        String fileName = item.getName();
        String contentType = item.getContentType();
        boolean isInMemory = item.isInMemory();
        long sizeInBytes = item.getSize();
        
        InputStream uploadedStream = item.getInputStream();
     
        uploadedStream.close();
    }
```



III. 如何得到 List<FileItem> 对象. 

```
    简单的方式

    // Create a factory for disk-based file items
    FileItemFactory factory = new DiskFileItemFactory();
    
    // Create a new file upload handler
    ServletFileUpload upload = new ServletFileUpload(factory);
    
    // Parse the request
    List /* FileItem */ items = upload.parseRequest(request);
    
    > 复杂的方式: 可以为文件的上传加入一些限制条件和其他的属性
    
    // Create a factory for disk-based file items
    DiskFileItemFactory factory = new DiskFileItemFactory();
    
    //设置内存中最多可以存放的上传文件的大小, 若超出则把文件写到一个临时文件夹中. 以 byte 为单位
    factory.setSizeThreshold(yourMaxMemorySize);
    //设置那个临时文件夹
    factory.setRepository(yourTempDirectory);
    
    // Create a new file upload handler
    ServletFileUpload upload = new ServletFileUpload(factory);
    
    //设置上传文件的总的大小. 也可以设置单个文件的大小. 
    upload.setSizeMax(yourMaxRequestSize);
    
    // Parse the request
    List /* FileItem */ items = upload.parseRequest(request);



```


这里        InputStream in = req.getInputStream();  的流只能读一次，


参考代码如下：
```

@WebServlet(name = "upLoadServlet", urlPatterns = {"/upLoadServlet"})
public class UpLoadServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //这里参考下面的常规方法。。。。。。

        // 这里开始使用upload 组件中的方法了，这里省去了我们自己来解析流，获得参数，文件内容等操作了。
        DiskFileItemFactory factory = new DiskFileItemFactory();
        factory.setRepository(new File("/tmp"));
        ServletFileUpload upload = new ServletFileUpload(factory);
        upload.setSizeMax(1024 * 1024 * 5);
        try {
            // 得到FileItem对象集合// 如果上面获取过流读取过流，这里的items集合是空的，这里要特别注意
            List<FileItem> items = upload.parseRequest(req); //这里调用解析请求的方法，获得一个FileItem 列表
            System.out.println("==" + items);
            //遍历, 判断是否是表单域，还是文件域
            for (FileItem item : items) {
                if (item.isFormField()) {
                    // 这里判断出来这个item是一个普通的域里面的参数
                    String name = item.getFieldName();
                    String value = item.getString();
                    System.out.println("=fileitem=" + name + "   " + value);
                    //　表单里面如果是有多选的话应该怎么办呢？

                } else {
                    // 这里判断出来参数是一个文件，也就是我们要上传给服务器的哪个文件
                    String fieldName = item.getFieldName();
                    String fileName = item.getName();
                    String contentType = item.getContentType();
                    boolean isInMemory = item.isInMemory();
                    long sizeInBytes = item.getSize();

                    System.out.println("fieldName    = " + fieldName);
                    System.out.println("fileName     = " + fileName);
                    System.out.println("contentType  = " + contentType);
                    System.out.println("isInMemory   = " + isInMemory);
                    System.out.println("sizeInBytes  = " + sizeInBytes);
                    
                    //这里就可以写上读取流，并把这个流在存储在服务器上硬盘中的某个文件啦。
                    // 参考下面的代码
                }//end if
            }//end for

            //若是文件域就把文件保存到某个目录下面
        } catch (FileUploadException e) {
            e.printStackTrace();
        }


    }
}


```

这里是按照常规方法取获取请求中参数，这个时候是获取不到的参数内容的，
这里可以获取请求中的一个InputStream流，通过这个流可以倒是可以获取请求中的参数，不过比较麻烦需要取解析的。

```
        String desc = req.getParameter("desc");
        String file = req.getParameter("file");
        System.out.println(desc);
        System.out.println(file);
        InputStream in = req.getInputStream();
        //这里的流只能读一次，下面的upload就读不到了

        BufferedReader reader = new BufferedReader(new InputStreamReader(in));
        String str = null;
        while ((str = reader.readLine()) != null) {
            System.out.println("==raw data ===" + str);
        }
        //这里不支持reset方法，应为没有重新父类的reset
        //in.reset();
```

这里我们读取FileItem中的输入流，把文件保存到服务器上面的/tmp目录，
这里我指定的是linux服务器上面的/tmp这样的一个临时目录。
```
                    //这里加上文件输入输出的操作来保存上传的文件到一个目录下面。
                    InputStream in = item.getInputStream();
                    byte[] buffer = new byte[1024];
                    int len = 0;
                    OutputStream out = new FileOutputStream("/tmp/" + fileName);
                    while ((len = in.read(buffer)) != -1) {
                        out.write(buffer);
                    }
                    in.close();
                    out.close();
```

表单里面如果是有多选的话应该怎么办呢？

```
<input type="checkbox" name="interesting" value="reading" />reading
<input type="checkbox" name="interesting" value="party" />party
<input type="checkbox" name="interesting" value="sports" />sports
<input type="checkbox" name="interesting" value="shopping" />shopping

for (FileItem item : items) {
      if (item.isFormField()) {
       // 这里判断出来这个item是一个普通的域里面的参数
          String name = item.getFieldName();
          String value = item.getString();
          System.out.println("=fileitem=" + name + "   " + value);
          //　表单里面如果是有多选的话应该怎么办呢？
          // 每一个都对应一个FileItem 对象，其中的getFieldName()都是interesting，可以直接拼出一个数组。
      }//end if()
}//end for()
```
如何修改工具类，框架的源代码？
１）原则：能不修改就不修改。
２）修改方法
    >修改源代码，替换ｊａｒ包中对应的ｃｌａｓｓ文件。
    >在本地新建相同的包，和类，在这个类里面修改即可。
        



----------


1). 需求:

I.  上传

    > 在 upload.jsp 页面上使用 jQuery 实现 "新增一个附件", "删除附件". 但至少需要保留一个.

    > 对文件的扩展名和文件的大小进行验证. 以下的规则是可配置的. 而不是写死在程序中的. 

        >> 文件的扩展名必须为 .pptx, docx, doc
        >> 每个文件的大小不能超过 1 M
        >> 总的文件大小不能超过 5 M.

    > 若验证失败, 则在 upload.jsp 页面上显示错误消息: 

        >> 若某一个文件不符合要求: xxx 文件扩展名不合法 或 xxx 文件大小超过 1 M
        >> 总的文件大小不能超过 5 M.

    > 若验证通过, 则进行文件的上传操作

        >> 文件上传, 并给一个不能和其他文件重复的名字, 但扩展名不变
        >> 在对应的数据表中添加一条记录. 

        id  file_name  file_path  file_desc


首先来使用jquery来把界面做出来
```
<head>
    <title>上传</title>
    <!-- 这里是引用jquery的路径，这里使用斜杠， -->
    <script type="text/javascript" src="/scripts/jquery-1.7.2.js"></script>

    <script type="text/javascript">
        $(function () {
            var i = 2;
            // 1. 获取#addFile,并为其添加click响应函数，这里首先完成添加的功能
            $("#addFile").click(function () {
                //注意这里的br标签加了个 id，不然这里的$("#br")好像获取不到
                $("#br").before(
                    '文件File ' + i + ': <input type="file" name="file' + i + '"/>'
                    + '<br/>'
                    + '描述Desc ' + i + ': <input type="text" name="desc' + i + '"/>'
                    + '<br/>'
                    + '<button id="delete">删除</button>'
                );
                
                // 这里别忘了 i 自增一下。
                i ++ ;
                
                // 需要注意的一个地方是这里的click方法要返回一个false，
                // 不然这个按钮在form表单中被点击的时候会触发提交表单的动作
                return false;

            });

            // 2. 利用jquery来生成新的,注意数字的变化，加到一个br标签前面，
            //    文件File 1: <input type="file" name="file1"/>
            //    <br/>
            //    描述Desc 1: <input type="text" name="desc1"/>
            //    <br/>
            //    <button id="delete">删除当前的file和desc的input节点</button>


        });


    </script>
</head>
```

下面是表单form中的代码

```
<form action="/upLoadServlet" method="post" enctype="multipart/form-data">
    
    <input type="hidden" id="fileNum" name="fileNum" value="1">
    
    <br/><br/>
    文件File: <input type="file" name="file"/>
    <br/><br/>
    Desc: <input type="text" name="desc"/>
    <button id="delete">删除当前的file和desc的input节点</button>
    <br/><br/>

    <!-- 注意这里的br标签加了个 id，不然上面的jquery好像获取不到，-->
    <br id="br">

    <br/><br/>
    <input type="submit" name="submit" value="submit"/>

    <!-- 按钮 需要js来支持 -->
    <button id="addFile">新增一个附件</button>


</form>
```

这里增加了一个删除的功能。用一个div把那些内容都包含进去，删除的时候直接删除这个div标签。
不过这个删除之后 带数字的地方都没有重新排序，下面会把它重新排序一下。
```
    <script type="text/javascript">
        $(function () {
            var i = 2;
            //获取#addFile,并为其添加click响应函数
            $("#addFile").click(function () {
                $("#br").before(
                    '<div>'
                    + '文件File ' + i + ': <input type="file" name="file' + i + '"/>'
                    + '<br/>'
                    + '描述Desc ' + i + ': <input type="text" name="desc' + i + '"/>'
                    + '<button id="delete">删除</button></div>'
                ).prev("div").find("button").click(function () {
                    $(this).parent("div").remove();
                    i--;
                    return false;
                });

                i++;
                return false;

            });

        });


    </script>
```


下面对代码进行优化，上面的代码input，button都是松散的，这里我们可以他们一起放到一个tr里面，这样就能很好的整体操作他们了。

```
    <script type="text/javascript">
        $(function () {
            var i = 2;
            $("#addFile").click(function () {
                $(this).parent().parent().before(
                    '<tr><td>File' + i + ':</td><td><input type="file" name="file' + i + '"/></td></tr>' +
                    '<tr><td>Desc' + i + ':</td><td><input type="text" name="desc' + i + '"/><button id="delete' + i + '">删除</button></td></tr>'
                );

                i++;
                //获取新添加的删除按钮
                $("#delete" + (i - 1)).click(function () {
                    var $tr = $(this).parent().parent();
                    $tr.prev("tr").remove();
                    $tr.remove();
                    i--;
                });

                return false;
            });

        });


    </script>
```

```
    <table>
        <tr>  <td>File1:</td> <td><input type="file" name="file"/></td>       </tr>
        <tr>  <td>Desc1:</td> <td><input type="text" name="desc"/></td>        </tr>
        <tr> <td> <input type="submit" name="submit" value="submit"/> </td> <td>  <!-- 按钮 需要js来支持 -->  <button id="addFile">新增一个附件</button> </td> </tr>
    </table>
```

删除功能基本上可以，但是还缺少一个对删除后的tr进行重新排序的功能，下面加上排序的功能


```
    <!-- 下面是js的代码 -->
    <script type="text/javascript">
        $(function () {
            var i = 2;
            $("#addFile").click(function () {
                $(this).parent().parent().before(
                    '<tr class="file"><td>File' + i + ':</td><td><input type="file" name="file' + i + '"/></td></tr>' +
                    '<tr class="desc"><td>Desc' + i + ':</td><td><input type="text" name="desc' + i + '"/><button id="delete' + i + '">删除</button></td></tr>'
                );

                i++;
                //获取新添加的删除按钮
                $("#delete" + (i - 1)).click(function () {
                    var $tr = $(this).parent().parent();
                    $tr.prev("tr").remove();
                    $tr.remove();

                    // 对i进行重新排序，这个就是删除了中间的某一个的时候，这个数字还是按照顺序的
                    // 下面的代码是进行重新排序的，也就是不管原来是什么顺序，什么数字，这里就是按照0,1,2,3，这样的顺序遍历，重新把数字设置上而已。
                    $(".file").each(function (index) {
                        var n = index + 1;
                        $(this).find("td:first").text("File" + (n));
                        $(this).find("td:last input").attr("name", "file" + (n));
                    });
                    $(".desc").each(function (index) {
                        var n = index + 1;
                        $(this).find("td:first").text("Desc" + (n));
                        $(this).find("td:last input").attr("name", "desc" + (n));
                    });


                    i--;
                });

                return false;
            });

        });


    </script>
    <!-- 上面是js的代码 -->

```

```
    <table> <!-- 这里把之前的input，button等标签都放到一个表格table中 -->
        <tr class="file"><td>File1 : </td><td><input type="file" name="file"/></td> </tr>
        <tr class="desc"><td>Desc1 : </td> <td><input type="text" name="desc"/></td></tr>
        <tr><td><input type="submit" name="submit" value="submit"/></td><td> <!-- 按钮 需要js来支持 --> <button id="addFile">新增一个附件</button></td></tr>
    </table>
```

下面我们来做个prop文件来配置之前需求中提到的几个配置，这些配置可以写到.prop属性配置文件中，这个比较简单，
复杂的可以写到xml配置文件中。这里我们采取简单的办法，写到一个upload.prop 属性文件中

    > 对文件的扩展名和文件的大小进行验证. 以下的规则是可配置的. 而不是写死在程序中的. 

        >> 文件的扩展名必须为 .pptx, docx, doc
        >> 每个文件的大小不能超过 1 M
        >> 总的文件大小不能超过 5 M.

uoload.prop 文件的内容
```
# 支持的文件的扩展名
exts=pptx,docx,doc

# 一个文件的大小
file.max.size=1048576

# 总的大小
total.max.size=5242880

# 这个要在servletlistener中读取，初始化

```
这个属性文件要在启动web应用的时候进行初始化，而且只需要初始化一次就够了。这时候我们应该能够想到
ServletContextListener的作用了。所有我们定义一个FileUploadAppListener的类来做这个web应用的初始化相关的操作。

```
public class FileUploadAppListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        //这里获取属性文件的输入流
        InputStream in = getClass().getClassLoader().getResourceAsStream("/upload.prop");

        Properties properties = new Properties(); // new出来一个Properties对象，下面就会调用其load（）方法了。
        try {
            properties.load(in);//通过上面获得的输入流，调用属性对象的load（）方法
            for (Map.Entry<Object, Object> prop : properties.entrySet()) {
                // 遍历所有属性，并把他们存到一个FileUploadAppProperties里里面，这个类采取了一个单例模式，
                // 这个类里面定义了一个map对象，也就是把属性名字，属性值都存在一个字典中。
                FileUploadAppProperties.getInstance().addProperty(
                        (String) prop.getKey(),
                        (String) prop.getValue()
                );//add prop
            }
        } catch (IOException e) {
            e.printStackTrace();
        }


        System.out.println("FileUploadAppListener: "+ FileUploadAppProperties.getInstance().getProperties());


    }

    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        //未使用的
    }
}


```
上面的listener代码写好之后，千万别忘记在web.xml中配置注册这个listener。

```
   <listener>
        <listener-class>com.test.mvcapp.listener.FileUploadAppListener</listener-class>
    </listener>
```




下面我们看这个存储属性的那个类，这个类很简单，里面定义了一个map用来存储属性。
这个类同时采用了单例设计模式。

```
public class FileUploadAppProperties {

    private Map<String, String> properties = new HashMap<>();

    private FileUploadAppProperties() {

    }

    private static FileUploadAppProperties instance = new FileUploadAppProperties();

    public static FileUploadAppProperties getInstance() {
        return instance;
    }


    public void addProperty(String key, String value) {
        properties.put(key, value);
    }


    public String getProperty(String key) {
        return properties.get(key);
    }

    public Map<String, String> getProperties() {
        return properties;
    }

}
```
属性可配置这个解决了，下面我们就来新建一个servlet，upLoadServlet
注意这里我们采用了注解的方式来配置这个servlet类和url的映射的。
首先我们拿到之前解析到的属性的配置的内容，先尝试的这打印出看看值对不对。

```
@WebServlet(name = "upLoadServlet", urlPatterns = {"/upLoadServlet"})
public class FileUploadServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        String exts = FileUploadAppProperties.getInstance().getProperty("exts");
        String fileMaxSize = FileUploadAppProperties.getInstance().getProperty("file.max.size");
        String totalMaxSize = FileUploadAppProperties.getInstance().getProperty("total.max.size");

        System.out.println("exts         = " + exts);
        System.out.println("fileMaxSize  = " + fileMaxSize);
        System.out.println("totalMaxSize = " + totalMaxSize);

        System.out.println("=============================================================");
        
        DiskFileItemFactory factory = new DiskFileItemFactory();
        factory.setSizeThreshold(1024*500);
        factory.setRepository(new File("/tmp"));
        ServletFileUpload upload = new ServletFileUpload(factory);
        upload.setFileSizeMax(Integer.parseInt(fileMaxSize));
        upload.setSizeMax(Integer.parseInt(totalMaxSize));

        // 后续操作文件上传的代码
        try {
            // 得到FileItem对象集合
            List<FileItem> items = upload.parseRequest(req);
            //......           
        } catch (FileUploadException e) {
            e.printStackTrace();
        }

    }//end doPost
}

```
这里把相关的好多代码放到一个单独的方法里面，这样doPost()方法看着会更简洁。
```
@WebServlet(name = "upLoadServlet", urlPatterns = {"/upLoadServlet"})
public class FileUploadServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
         // 这里直接把相关的好多代码放到一个单独的方面里面，这样ｄｏＰｏｓｔ（）方法看着会更简洁。
         ServletFileUpload upload = getServletFileUpload();
         
        // 后续操作文件上传的代码
        try {
            // 得到FileItem对象集合
            List<FileItem> items = upload.parseRequest(req);
            //......           
        } catch (FileUploadException e) {
            e.printStackTrace();
        }
    }//end doPost

    private ServletFileUpload getServletFileUpload() {
        //在ｉｄｅａ中选中相应的代码块，然后　选择 Refactor | Extract | Method 或者 按 Ctrl+Alt+M
        
        String exts = FileUploadAppProperties.getInstance().getProperty("exts");
        String fileMaxSize = FileUploadAppProperties.getInstance().getProperty("file.max.size");
        String totalMaxSize = FileUploadAppProperties.getInstance().getProperty("total.max.size");
        
        DiskFileItemFactory factory = new DiskFileItemFactory();
        factory.setSizeThreshold(1024 * 500);
        factory.setRepository(new File("/tmp"));
        ServletFileUpload upload = new ServletFileUpload(factory);
        upload.setFileSizeMax(Integer.parseInt(fileMaxSize));
        upload.setSizeMax(Integer.parseInt(totalMaxSize));
        return upload;
    }
}

```
下面我们来完成上传文件的相关的代码，这个代码可以参考之前上面的那个例子，很大部分代码是重复的。
这里 上传文件相关的 有文件名称，文件描述，还有文件路径，文件id等，这个时候我们需要考虑写一个
关于文件信息的javabean类了
```

public class FileUploadInfo implements Serializable {
    //文件的信息相关的一个javabean类
    private int id;
    private String fileName;
    private String filePath;
    private String fileDesc;

    public FileUploadInfo() {//无参数的构造器，必须的。为了反射使用的
    }

    public FileUploadInfo(String fileName, String filePath, String fileDesc) {
        this.fileName = fileName;
        this.filePath = filePath;
        this.fileDesc = fileDesc;
    }

    //　下面是一些getter，setter方法，这些方法使用idea都会自动生成的。
    public int getId() {}
    public void setId(int id) {}
    public String getFileName() {}
    public void setFileName(String fileName) {}
    public String getFilePath() {}
    public void setFilePath(String filePath) {}
    public String getFileDesc() {}
    public void setFileDesc(String fileDesc) {}
    public boolean equals(Object o) {}
    public int hashCode() {    }
    public String toString() {    }
}


```
我们来完成上传文件的try　catch里面的一些步骤，把每一个小步骤都定义为一个简单的独立的方法，这样try语句块就看着比较简单，清晰。

```
        try {
            // 得到FileItem对象集合
            List<FileItem> items = upload.parseRequest(req);

            //填充FileUploadInfo的集合,同时填充这个集合uploadFiles
            Map<String, FileItem> fileUploadMap = new HashMap<>();
            List<FileUploadInfo> fileUploadInfoList = new ArrayList<>();
            //填充bean集合
            fillBeans(items, fileUploadInfoList, fileUploadMap);

            //校验文件扩展名
            vaidateExtName(fileUploadInfoList);

            //校验文件大小,这个不需要直接写，这个在解析时候已经校验了，如果大小不符合会抛出异常的

            //以上都通过就进行文件的上传操作
            uploadFile(fileUploadMap);

            //把上传成功的文件的信息保存到数据库中
            saveBeans(fileUploadInfoList);
        } catch (FileUploadException e) {
            e.printStackTrace();
        }
```
抽象出来的方法，这里我们还没有具体的代码，稍后会补上具体的代码的。
```

    private void saveBeans(List<FileUploadInfo> beans) {
    }

    private void uploadFile(Map<String, FileItem> uploadFiles) {
    }

    private void vaidateExtName(List<FileUploadInfo> beans) {
    }

    private void fillBeans(List<FileItem> items,  List<FileUploadInfo> fileUploadInfoList,  Map<String, FileItem> uploadFilesMap) {
        // 这个方法就是用来填充ｂｅａｎ的两个结合的，通过遍历ｉｔｅｍｓ然后分别把相应的内容填充到ｂｅａｎ集合中去。
    }

```
下面我们来完成这个fillBeans（）方法

```
    private void fillBeans(List<FileItem> items, List<FileUploadInfo> list, Map<String, FileItem> map) {
        //1. 遍历items集合，先得到desc的ｍａｐ，其中键是 FieldName 是表单域中的desc1，desc2等
        Map<String, String> descMap = new HashMap<>();
        for (FileItem item : items) {
            if (item.isFormField())
                descMap.put(item.getFieldName(), item.getString());
        }
        for (FileItem item : items) {
            if (!item.isFormField()) {
                //文件的这个表单域，file1,file2,file3,.....
                String fieldName = item.getFieldName(); //下面取到最后一个，最后一个是一个索引，一个数字，这个和ｄｅｓｃ是一一对应的
                String index = fieldName.replace("file", "");
                String fileName = item.getName();
                String desc = descMap.get("desc" + index);
                String filePath = getFilePath(fileName);
                FileUploadInfo info = new FileUploadInfo(fileName, filePath, desc);
                list.add(info);
                map.put(filePath, item);
            }
        }

    }//end fillBeans()


    private String getFilePath(String fileName) {
        //根据给定的文件名构建一个随机的文件名称，文件扩展名保持和原来的一样。
        String extName = fileName.substring(fileName.lastIndexOf("."));//获取文件的扩展名
        Random random = new Random();
        int randomNum = random.nextInt(1000000);
        String filePath = getServletContext().getRealPath(ROOT_PATH) + "/" + System.currentTimeMillis() + randomNum + extName;
        return filePath;
    }
```

下面我们来完成这个uploadFile（）方法,这个方法用来上传文件，保存文件到一个目录下面。
就是读取输入流，写入到输出流。

```

    private void uploadFile(Map<String, FileItem> map) {
        for (Map.Entry<String, FileItem> entry : map.entrySet()) {
            String filePath = entry.getKey();
            FileItem fileItem = entry.getValue();
            try {
                InputStream inputStream = fileItem.getInputStream();
                OutputStream outputStream = new FileOutputStream(filePath);
                byte[] buffer = new byte[10240];
                int len = 0;
                while ((len = inputStream.read(buffer)) != -1) {
                    outputStream.write(buffer);
                }
                inputStream.close();
                outputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }//end for()
    }
```
下面完成扩展名验证的方法，判断扩展名不合法就抛出一个InvalidExtNameException异常，这个异常需要我们自己定义一个异常类。
只要有一个不符合就都不允许上传，这里抛出一个InvalidExtNameException异常。

```
    private void vaidateExtName(List<FileUploadInfo> list) {
        String exts = FileUploadAppProperties.getInstance().getProperty("exts");
        List<String> extList = Arrays.asList(exts.split(","));
        for (FileUploadInfo fileUploadInfo : list) {
            String fileName = fileUploadInfo.getFileName();
            String extName = fileName.substring(fileName.lastIndexOf(".") + 1);

            if (!extList.contains(extName)) {
                //只要有一个不符合就都不允许上传，这里抛出一个异常
                throw new InvalidExtNameException("文件的扩展名不合法：" + fileName);
            }
        }
    }
```
我们自己定义一个异常InvalidExtNameException类。
异常类一般继承RuntimeException这个父类。
```
public class InvalidExtNameException extends RuntimeException {

    public InvalidExtNameException() {
    }

    public InvalidExtNameException(String message) {
        super(message);
    }
}
```
这样在doPost()　方法里面我们就要取捕获这个异常了。
这里我们定义一个转发的path，上传成功转发到一个页面，失败的话还回到upload.jsp面。
```
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        ServletFileUpload upload = getServletFileUpload();

        String path;
        try {
            // 得到FileItem对象集合
            List<FileItem> items = upload.parseRequest(req);

            //.......省略其他代码
            
            //校验文件扩展名
            vaidateExtName(fileUploadInfoList);

            //.......省略其他代码
            
            path = "/updown/success.jsp";
        } catch (Exception e) {
            e.printStackTrace();
            path = "/updown/upload.jsp";
            req.setAttribute("message", e.getMessage());
        }

        req.getRequestDispatcher(path).forward(req, resp);

    }//end doPost
    
```
接下来完成保存上传文件信息到数据库saveBeans()　这个方法。
代码很简单直接调用DAO对象里面的saveFiles方法。这里说到关于DAO相关的类，用法可以参考稍后的总结。
这个个DAO是一个类成员变量，是FileUploadInfoDAO类型的对象。
```
    private FileUploadInfoDAO dao = new FileUploadInfoDAOImpl();
    //....　省略中间的部分代码
    private void saveBeans(List<FileUploadInfo> list) {
        dao.saveFiles(list);
    }
```
关于FileUploadInfoDAO这个接口的定义：
```
public interface FileUploadInfoDAO {
    List<FileUploadInfo> getFiles();
    void saveFiles(List<FileUploadInfo> list);
    void saveFile(FileUploadInfo fileUploadInfo);
}
```
然后是这个接口的一个具体实现类，也就是我们上面new出来的那个对象的类
这个类实现了FileUploadInfoDAO接口，并且继承了一个DAO的父类。
```
public class FileUploadInfoDAOImpl extends DAO<FileUploadInfo> implements FileUploadInfoDAO {
    @Override
    public List<FileUploadInfo> getFiles() {
        String sql = "SELECT id, file_name fileName, file_path filePath, "
                + "file_desc fileDesc FROM upload_files";
        return getForList(sql);
    }

    @Override
    public void saveFiles(List<FileUploadInfo> list) {
        for (FileUploadInfo file : list) {
            saveFile(file);
        }
    }

    @Override
    public void saveFile(FileUploadInfo file) {
        String sql = "INSERT INTO upload_files (file_name, file_path, file_desc) VALUES (?, ?, ?)";
        update(sql, file.getFileName(), file.getFilePath(), file.getFileDesc());
    }
}
```
这里是DAO这个父类里面代码，这个里面主要就是关于数据库的方法，常规的增删改查这些方法。
```
public class DAO<T> {
    private QueryRunner queryRunner = new QueryRunner();
    private Class<T> clazz;

    public DAO() {
        Type superClass = getClass().getGenericSuperclass();
        if (superClass instanceof ParameterizedType) {
            ParameterizedType parameterizedType = (ParameterizedType) superClass;

            Type[] typeArgs = parameterizedType.getActualTypeArguments();
            if (typeArgs != null && typeArgs.length > 0) {
                if (typeArgs[0] instanceof Class) {
                    clazz = (Class<T>) typeArgs[0];
                }
            }
        }
    }


    public <E> E getForValue(String sql, Object... args) {
        //获取一个的
        Connection connection = null;
        try {
            connection = JdbcUtils.getConnection();
            return (E) queryRunner.query(connection, sql, new ScalarHandler(), args);
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.releaseConnection(connection);
        }
        return null;
    }

    public List<T> getForList(String sql, Object... args) {
        //获取一个的
        Connection connection = null;
        try {
            connection = JdbcUtils.getConnection();
            return queryRunner.query(connection, sql,
                    new BeanListHandler<>(clazz), args
            );
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.releaseConnection(connection);
        }
        return null;
    }


    public T get(String sql, Object... args) {
        System.out.println(clazz);
        //获取一个的
        Connection connection = null;
        try {
            connection = JdbcUtils.getConnection();
            return queryRunner.query(connection, sql, new BeanHandler<T>(clazz), args);
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.releaseConnection(connection);
        }
        return null;
    }

    public void update(String sql, Object... args) {
        Connection connection = null;
        try {
            connection = JdbcUtils.getConnection();
            queryRunner.update(connection, sql, args);
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.releaseConnection(connection);
        }

    }

}
```

这里DAO类里面数据库的链接等使用到了一个jdbcutil这个类里面的static方法。获取数据库链接，释放数据库链接都是调用的JdbcUtils类里面的方法。
```
public class JdbcUtils {
    private static DataSource dataSource = null;
    static {
        dataSource = new ComboPooledDataSource("mvcapp");
    }

    public static void releaseConnection(Connection connection) {
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }

}
```

这其中ComboPooledDataSource类是c3p0包里面的，这里使用到了c3p0数据库相关的包，这里有个c3p0相关的xml配置文件配置文件主要就是定义了数据库名称，数据库账户密码等。
如果出现中文编码的问题可以把url那里改为
       `        <property name="jdbcUrl">jdbc:mysql:///test?useUnicode=true&amp;characterEncoding=UTF8 </property>` 

```
<c3p0-config>

    <!-- This app is massive! -->
    <named-config name="mvcapp">
        <property name="user">test</property>
        <property name="password">123456</property>
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql:///test</property>


        <property name="acquireIncrement">5</property>
        <property name="initialPoolSize">10</property>
        <property name="minPoolSize">5</property>
        <property name="maxPoolSize">10</property>

        <!-- intergalactoApp adopts a different approach to configuring statement caching -->
        <property name="maxStatements">10</property>
        <property name="maxStatementsPerConnection">5</property>

        <!-- he's important, but there's only one of him -->
        <user-overrides user="master-of-the-universe">
            <property name="acquireIncrement">1</property>
            <property name="initialPoolSize">1</property>
            <property name="minPoolSize">1</property>
            <property name="maxPoolSize">5</property>
            <property name="maxStatementsPerConnection">50</property>
        </user-overrides>
    </named-config>
</c3p0-config>
```

----------
##文件下载

1.设置 contentType 响应头: 设置响应的类型是什么 ?   通知浏览器是个下载的文件

```
response.setContentType("application/x-msdownload"); 
```


2.设置 Content-Disposition 响应头: 通知浏览器不再由浏览器来自行处理(或打开)要下载的文件, 而由用户手工处理

```
response.setHeader("Content-Disposition", "attachment;filename=abc.txt");
```


2.具体的文件: 可以调用 response.getOutputStream 的方式, 以 IO 流的方式发送给客户端.

```
OutputStream out = response.getOutputStream();
String pptFileName = "/tmp/JavaWEB_监听器.pptx";

InputStream in = new FileInputStream(pptFileName);

byte [] buffer = new byte[1024];
int len = 0;

while((len = in.read(buffer)) != -1){
    out.write(buffer, 0, len);
}

in.close();
```




