###OSX安装JAVA开发环境

@(马克飞象)[java, osx, 系统, 环境, 搭建]


>**一般OSX会自带JAVA环境，但版本可能就会比较老了**

#### 检查系统java版本

    java -version
    //本机系统java版本
    java version "1.7.0_79"
    Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
    Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)

#### JDK安装和环境变量配置
1. 选择自己对应的系统版本 [官网下载地址](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 
2. 查看环境变量
	```
	cat ~/.bash_profile
	```
	如果已经设置的话会显示出来

3. 设置环境变量
	```
	sudo vim ~/.bash_profile
	```
	>type i 进入编辑模式，输入：
	```
	# 设置JAVA环境变量，路径根据自己的安装有所不同
	JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.7.0_79.jdk/Contents/Home/"
	CLASSPATH="$JAVA_HOME/lib"
	PATH=".;$PATH:$JAVA_HOME/bin"
	export JAVA_HOME
	export CLASSPATH
	```
4. 验证 
	> 注：**新开一个终端**
```
echo $JAVA_HOME
```
> 如果正确，则会显示上一步设置的路径
> 如果不正确，则请重复检查之前的步骤

#### 安装Tomcat
> 介绍：动态服务器，一般与Nginx配合使用，Nginx负责静态数据，tomcat负责动态部分。
> [官网下载](http://tomcat.apache.org/)

1. 选择**Core zip**格式即可，解压到**/Library/Java/Tomcat/**目录下
2. 进入bin目录下查看sh脚本 
```
// 查看所有.sh脚本权限【bin目录】
ls -la *.sh

// 修改所有sh脚本权限【bin目录上级】
chmod -R u+x ./bin
```
如果修改成功的话可以**ls -la *.sh**看到权限的变化
3. 启动
```
//进入bin目录输入
./startup.sh
```
> 成功的话会看到提示：**Tomcat started.**
> 浏览器中输入http://localhost:8080/ 就可以看到🐅和🐈的画面了

#### 安装Maven
> JAVA工程中的配置系统，**类似于XCode工程中的Pod管理器**。
> [官网下载地址](http://maven.apache.org/download.cgi)选择bin.zip即可

1. 下载后解压至系统Java目录下
2. 修改环境变量
```
sudo vim ~/.bash_profile
```
123
>增加/修改如下代码：
>
	# 设置JAVA环境变
	MAVEN_HOME="/Library/Java/apache-maven-3.3.9"
	PATH=".;$PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin"
	export MAVEN_HOME


3. 测试
```
mvn -v
```
如果正确输出版本信息即证明安装成功

#### Eclipse安装
> JAVA 开发环境，因为是开发JSP所以选择J2EE
> [官网下载地址](http://www.eclipse.org/downloads/packages/eclipse-ide-java-ee-developers/keplersr2)

1. 双击安装
2. 启动，配置JRE环境，`Preferences->Java->Installed JREs`选择本机安装的Java版本，如果没有的话需要添加这个路径，这里的要和你之前配置的环境变量的Java版本一致。
3. 关联Tomcat，`Preferences->Server->Runtime Envionments`，`Add->Apache`，选择自己本机安装的Tomcat版本，我们这里选择v7.0。
4. 关联Maven，`Preferences->Maven->Installations`，`Add->选择Maven本机安装的目录`

#### 导入工程
> 本地新建工程，下载工程
> 开始导入：
> `Import->Maven->Existing Maven Projects->选择工程文件夹`


#### SVN绑定
`Help->Install New Software...->Add...`
> 输入插件地址：http://subclipse.tigris.org/update_1.8.x
> 一路`next`等待安装完成

左侧边工程项目右键 `Team->Share Projects->`
输入svn地址，配置好账号密码，成功后即可在每个项目右键`Team->更新`