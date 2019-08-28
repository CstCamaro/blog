## weblogic 10.3.6 环境搭建

### 环境准备
#### 操作系统
Ubuntu 16.04LTS
#### weblogic 10.3.6 安装包
http://download.oracle.com/otn/nt/middleware/11g/wls/1036/wls1036_generic.jar

### 安装步骤
1、升级apt
```
sudo apt-get update && apt-get upgrade
```

2、安装Java环境
```
sudo apt-get install openjdk-8-jdk
```
安装完成后，查看Java版本，是否安装成功
```markdown
camaro@ubuntu:~$ java -version
openjdk version "1.8.0_222"
OpenJDK Runtime Environment (build 1.8.0_222-8u222-b10-1ubuntu1~16.04.1-b10)
OpenJDK 64-Bit Server VM (build 25.222-b10, mixed mode)
```

3、配置hosts
在你的hosts文件中添加以下行
```
127.0.0.1      localhost localhost.localdomain localhost4 localhost4.localdomain4
```

4、创建新用户
```
sudo groupadd -g 1001 oinstall
sudo useradd -u 1100 -g oinstall oracle
sudo passwd oracle
```

5、选择你所需要安装的目录
```
sudo mkdir -p /u01/app/oracle/product/fmw11g
sudo mkdir -p /u01/app/oracle/config/domains
sudo mkdir -p /u01/app/oracle/config/applications
sudo chown -R oracle:oinstall /u01
sudo chmod -R 775 /u01/
```

6、在/home/oracle/下创建.bash_profile文件，并将以下内容添加到.bash_profile文件中
```
export MW_HOME=/u01/app/oracle/product/fmw11g
export WLS_HOME=$MW_HOME/wlserver_10.3
export WL_HOME=$WLS_HOME
# Set to the appropriate JAVA_HOME.
#export JAVA_HOME=/usr/java/jdk1.6.0_33
export JAVA_HOME=/u01/app/oracle/jrockit-jdk1.6.0_45-R28.2.7-4.1.0
#export JAVA_HOME=/u01/app/oracle/jdk1.7.0_17
export PATH=$JAVA_HOME/bin:$PATH
```
7、将下载好的wls1036_generic.jar复制到自定义文件夹下(任意目录即可)，执行以下命令启动安装程序
```
java -Xmx1024m -jar wls1036_generic.jar
```
![weblogic_install_step_1.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_install_step_1.jpg)
8、等待弹出安装界面后点击下一步，进入Choose Middleware Home Directory
在Middleware Home Directory输入框内输入我们刚才配置的安装目录```/u01/app/oracle/product/fmw11g```，点击下一步

![weblogic_install_step_2.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_install_step_2.jpg)

9、这里勾不勾选都可以，点击下一步
![weblogic_install_step_3.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_install_step_3.jpg)

10、不设置代理，如图所示，点击下一步
![weblogic_install_step_4.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_install_step_4.jpg)

11、选择标准安装，点击下一步
![weblogic_install_step_5.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_install_step_5.jpg)

12、选择JDK版本，点击下一步
![weblogic_install_step_6.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_install_step_6.jpg)

13、选择安装目录，点击下一步
![weblogic_install_step_7.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_install_step_7.jpg)

14、选择安装内容，没有特殊需求的默认全部选择
![weblogic_install_step_8.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_install_step_8.jpg)

15、安装完成，可以使用Quick Start配置
![weblogic_install_step_9.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_install_step_9.jpg)

### weblogic Quick Start
1、打开weblogic Quick Start，选择Getting start with Weblogic Server@10.3.6
![weblogic_quick_start_step_1.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_quick_start_step_1.jpg)

2、创建新域名
![weblogic_quick_start_step_2.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_quick_start_step_2.jpg)

3、选择域名类型
![weblogic_quick_start_step_3.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_quick_start_step_3.jpg)

4、自定义域名名称
![weblogic_quick_start_step_4.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_quick_start_step_4.jpg)

5、设置weblogic登录时的账号密码，在以后登录时候需要用到，密码需要符合强度规范，字母+数字
![weblogic_quick_start_step_5.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_quick_start_step_5.jpg)

6、选择JDK，默认就行
![weblogic_quick_start_step_6.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_quick_start_step_6.jpg)

7、这一步看个人需求，默认都没有勾选
![weblogic_quick_start_step_7.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_quick_start_step_7.jpg)

8、点击下一步，等待安装完成即可
![weblogic_quick_start_step_8.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_quick_start_step_8.jpg)

### 启动weblogic
1、进入我们刚才安装的weblogic目录，此时可能会出现没有权限访问文件或文件夹的问题，修改文件或文件夹权限即可
```
cd /u01/app/oracle/product/fmw11g/user_projects/domains/camaro
./startWebLogic.sh
```

2、打开浏览器，在地址栏访问
```
http://localhost:7001/console
```
第一次访问时，可能会出现以下界面，稍等一会儿后刷新即可
![weblogic_first_access.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_first_access.jpg)
当出现以下界面时，weblogic部署已经完成
![weblogic_login.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_login.jpg)
