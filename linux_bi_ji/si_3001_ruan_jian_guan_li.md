## 四、软件管理

### 源码包安装

下载源代码安装包文件

>步骤1：tar包解压缩
   用途：解压并释放源代码包到指定的目录/usr/src/
步骤2：./configure 配置
   用途：设置安装目录、安装模块等选项(/usr/src/httpd-2.4.6/)
步骤3：make 编译
   用途：生成可执行的二进制文件Makefile
步骤4：make install   //安装
   用途：复制二进制文件到系统，配置应用环境并使用应用软件
    make uninstall    //卸载
    make clean        //清除之前编译的可执行文件与配置



举例：

>1. 将已下载的压缩包拷贝到本地，解压缩到指定路径`/usr/src`
        tar xf httpd-2.2.15.tar.gz -C /usr/src/

>2. 在解压目录中找到安装程序configure
        配置安装路径，配置安装模块，以及需要加载的功能
        ./configure --help 
        ./configure --prefix=/usr/local/httpd
        安装后会在当前目录下生成Makefile文件

>3. 对源包文件Makefile进行编译转换，转为系统能识别的二进制文件 
        make
>4. 将生成的二进制文件拷贝到安装目录中
        make install    //安装
>5. 测试：进入安装目录`/usr/local/httpd/bin/`执行`./httpd`开启httpd程序
>6. 可以通过网络监听指令获取程序运行信息
    netstat -lntup | grep :80



打开firefox，访问127.0.0.1 修改默认主站点信息，可以编辑/usr/local/httpd/htdocs/index.html 修改文件内容后刷新firefox，可看到更新内容

### rpm指令管理

#### 已安装：

```
rpm -qa 查询已安装的RPM包，列出系统中安装包的版本及相关信息
rpm -qi 软件名  列出已安装软件的详细信息
rpm -qf 文件/指令/目录  查看该文件或目录是由哪个RPM包所提供的

```

#### 未安装：

```
rpm -qpi 包.rpm  选项p表示package，后面需要写包的完整信息，qpi表示查询未安装包的详细信息
rpm -qpl 包.rpm  列出该rpm在安装后生成哪些文件和目录
安装与卸载：
rpm -ivh 包.rpm  i是安装，v显示安装进度，h以‘#’作为进度条，显示安装过程
rpm -e 软件名  移除指定的RPM包 

```

### yum软件管理

#### yum仓库源文件需要满足两点要求

1.  文件必须存放在/etc/yum.repos.d/目录中
2.  文件名称结尾必须以‘.repo’命名

#### repo文件中格式：

```
[base]             //中括号名称为yum的仓库源名称，通常为字母和数字
name=my new repo   //表示描述，用于方便管理员查看当前仓库源配置描述信息

//baseurl表示声明yum可以管理使用的rpm包路径，可以是基于网络的，也可以基于本地
baseurl=http://www.classroom.com/rhel7.0
baseurl=file:///mnt/cdrom  

enabled=1/0    //表示是否开启当前repo源，1表示开启，0表示关闭，此项省略，默认为开启
gpgcheck=0/1   //表示安装包的时候，是否基于公私钥对匹配包的安全信息，1表示开启匹配，0表示关闭验证，此项省略，默认为开启验证

```

#### yum命令

```
yum clean all          //清空缓存信息
yum list [包的名称]     //列出所有repo能使用的包[特定的rpm包]
yum install 包的名称 [-y]  //安装指定的rpm包
yum remove 包的名称 [-y]   //卸载指定的rpm包
yum search 关键词         //根据关键词，在已发现的repo源中搜索关键词关联的rpm包
yum provides 指令        //根据指令，在已发现的repo源中搜索提供指令的rpm包
yum history info/redo/undo  number   //根据之前的历史操作，yum执行查看，重装，反向安装等行为

```