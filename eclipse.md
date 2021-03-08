# Eclipse

## 指定 vm

* Mac 上编辑 eclipse.ini

  ```shell
  vi /Applications/Eclipse.app/Contents/Eclipse/eclipse.ini
  # 找到 -vmargs 所在行, 在这行上面添加2行
  -vm
  /usr/local/opt/openjdk/bin/java
  -vmargs

  ```

## 镜像

[参考][https://lug.ustc.edu.cn/wiki/mirrors/help/eclipse]

* 菜单: Performance -> Install/Update -> Avaliable Soft Site -> 将其中的 download.eclipse.org 替换为 mirrors.ustc.edu.cn/eclipse, 或者 mirrors.tuna.tsinghua.edu.cn/eclipse

## 插件

* Spring Tools: spring 套装工具

* Vrapper(vim): vim

* eclemma: 单元测试覆盖率工具

* sonarlint: 代码管理插件

* p3c: 阿里编码规范插件. 安装方法: 菜单 -> install new software -> https://p3c.alibaba.com/plugin/eclipse/update/

* easyshell: 打开 shell

* maven devlopement tools: m2e 的扩展

* subclipse: svn 插件

* JRebel: 生产力工具，它允许开发人员立即重新加载代码更改. 不免费.

* color ide pack: 颜色主题包

* autodetect encoding: 快速更改文件编码

* Relative Line Number Ruler: 显示相对行数的插件

* lombok: 通过注释减少代码量的工具. 下载: https://projectlombok.org/download . 手动安装: 拷贝到 /Applications/Eclipse.app/Contents/Eclipse/ 目录下. 然后在 同目录的 eclipse.ini 中添加以下两行.
  ```shell
  echo "-javaagent:/Applications/Eclipse.app/Contents/Eclipse/lombok.jar" >> eclipse.ini
  ```

* findbugs: 

* MyBatis Generator: 

* MyBatipse: mybatis 工具

* Enhanced Class Decompiler: 反编译工具

* Start Explorer: 打开文件浏览器

* developer utils: 开发小工具合集. regex, base64, md5 等

* Eclipse Archive Util: 在 eclipse 中打开 jar, war, zip 等