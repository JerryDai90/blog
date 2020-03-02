# NISS 使用管理员运行
我需要执行一个命令，但是这个命令需要管理员权限才可以运行。

```
Name "${PRODUCT_NAME} ${PRODUCT_VERSION}"
OutFile "xxxxx"
InstallDir "xxxxx"
InstallDirRegKey HKLM "${PRODUCT_UNINST_KEY}" "UninstallString"
ShowInstDetails show
ShowUnInstDetails show
RequestExecutionLevel admin
```
加入 RequestExecutionLevel admin 这句，在执行 mklink 的时候如下：

```
ExecWait 'cmd /c mklink /D "D:\test02" "D:\test01"'
```
注意：运行 `exe` 的时候需要右键使用管理员运行。

## 参考资料
https://nsis-dev.github.io/NSIS-Forums/html/t-314525.html
http://www.it610.com/article/1835566.htm



