## weblogic环境搭建

### 环境准备
#### 操作系统
Ubuntu 16.04LTS
#### weblogic 10.3.6 安装包
http://download.oracle.com/otn/nt/middleware/11g/wls/1036/wls1036_generic.jar

### 安装步骤
1、先升级apt
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
8、等待弹出安装界面后点击Next按钮，进入Choose Middleware Home Directory
在Middleware Home Directory输入框内输入我们刚才配置的安装目录```/u01/app/oracle/product/fmw11g```，点击下一步

![weblogic_install_step_2.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_install_step_2.jpg)

9、勾不勾选都可以，点击下一步
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

15、安装完成，可以使用Quick Start配置。
![weblogic_install_step_9.jpg](https://cstcamaro.github.io/blog/resource/img/weblogic_install_step_9.jpg)
