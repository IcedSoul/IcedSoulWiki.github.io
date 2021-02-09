# Maven安装使用问题记录

`Maven`是老朋友了，自从大三了解到这个工具之后一直使用至今，在使用的过程中当然也遇到了形形色色的问题，过往的问题现在也暂且不深究了，以后遇到的`Maven`相关的问题我会记录在这个文档中以便以后排查。



## Mac Maven Java版本问题

### 表象

最近刚使用Mac，使用`brew install maven`安装`Maven`之后，拉取依赖时报了错：

```shell
Fatal error compiling: java.lang.ExceptionInInitializerError: com.sun.tools.javac.code.TypeTags -> [Help 1]
```

查了一下发现这个报错和`Lombok`以及`Maven`配置的Java版本不匹配造成的，使用`mvn -v`命令查看后发现，虽然`Mac`配置的`Java`版本是jdk1.8，但是Maven配置的JVM是open jdk 1.15，也就是说使用brew install maven时自己下载了openjdk 1.15并且完成了配置，这并不会影响本身jdk1.8的Java配置。

### 原因

出现这个问题的主要原因是我使用了MAC自带的Java安装方法，它只在/etc/paths中添加了java路径，但是并没有配置JAVA_HOME的环境变量，在使用brew install maven时没有发现JAVA_HOME环境变量，就自行下载安装了openjdk，导致上述现象出现。

### 解决

解决这个问题的方法也不麻烦。

```shell
//先卸载不需要的openjdk
brew uninstall openjdk

//添加环境变量，MAC OS中环境变量有/etc/profile,/etc/paths,~/.bash_profile(~/.zshenv),~/.bash_login,~/.profile,~/.bashrc多种级别，自己决定选择合适的级别进行配置。下面语句为在～/.bash_profile末尾追加一行配置（若没有文件则新建）
//注意，需要根据自己的MAC OS版本确定写入哪个文件，如果1.15之前，应该把~/.zshenv替换~/.bash_profile
echo 'export JAVA_HOME=$(/usr/libexec/java_home)'>>~/.zshenv

//查看确认一下修改后的环境变量是否正确
cat ~/.zshenv

//应用修改后的环境变量
source ~/.zshenv

//查看JAVA_HOME环境变量的值
echo $JAVA_HOME

//确认mvn正常
mvn -v

```

### 参考资料

1. https://stackoverflow.com/questions/1348842/what-should-i-set-java-home-environment-variable-on-macos-x-10-6
2. https://www.jianshu.com/p/acb1f062a925
3. https://mkyong.com/java/how-to-set-java_home-environment-variable-on-mac-os-x/

