# Github搭建自己的Maven仓库

Maven 是一个版本依赖工具，它有一个官方的中央仓库，很多开源项目的 jar 包版本都能够在上面找到，但是将项目发布 jar 到中央仓库是在太麻烦了，因此我想到了用自己的 Github 作为仓库，从 Github 仓库中拉取依赖包。

## 搭建

在 GitHub 创建一个名为 Maven 的仓库，Git 地址：https://github.com/vinophantom/maven.git

进入本地 Maven 仓库，将仓库初始化为一个 Git 仓库，并添加远程仓库地址：

```shell
cd ~/.m2/repository
git init
git remote add origin https://github.com/vinophantom/maven.git
```

进入项目根目录中打包生成 .jar 文件，并部署到本地 Maven 仓库：

```shell
cd /Users/zhangchenghui/Documents/WXPaySDKJava
mvn package
cd target

mvn install:install-file -Dfile=WXPay-SDK-Java-0.0.5.jar -DgroupId=com.objcoding -DartifactId=WXPay-SDK-Java -Dversion=1.0.0 -Dpackaging=jar
```

将本地 Maven 仓库对应项目的文件提交到 GitHub：

```shell
cd ~/.m2/repository
git add -f com/objcoding/WXPay-SDK-Java/0.0.5
git commit -m 'WXPay-SDK-Java 0.0.5'
git push
```

这时会遇到本地 git 仓库和远程仓库没有关联的警告，输入一下命令可以解决：

```shell
git push --set-upstream origin master
```

## 使用

在项目 pom.xml 文件中加入如下内容：

```xml
<repositories>
    <repository>
      <id>objcoding-maven-master-repository</id>
      <name>objcoding-maven-master-repository</name>
      <url>https://raw.github.com/vinophantom/maven/master/</url>
    </repository>
  </repositories>

```

 