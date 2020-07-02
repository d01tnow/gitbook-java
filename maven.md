# maven

## 安装

mac 下安装: brew install maven

## 仓库

### ali 仓库

找到 maven 的配置文件 settings.xml. 命令 mvn -v 会显示. mac 位置: find /Applications/ -name settings.xml -exec vi {} \;
在 mirrors 标签下添加如下信息:

```xml
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

