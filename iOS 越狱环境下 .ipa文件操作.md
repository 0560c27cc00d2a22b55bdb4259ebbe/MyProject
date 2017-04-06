
### 解压+签名+打包+安装
#### 解压 
1. 新建文件夹  
2. 助手下载越狱版本 `xyz.ipa`  
3. 打开终端执进入文件夹，行命令 `unzip /path/xyz.ipa`  
<font color=red> 注：`xyz.app` 文件位于 `Payload` 文件夹中 </font>

####   签名
1. 执行命令 `security find-identity -v -p codesigning`，获取电脑上所有的签名证书  
2. 执行命令 `codesign -f -s "iPhone Distribution: xxxxxxxxxxxxxxxxx" xyz.app` 重新签名 

#### 打包 
执行命令 `zip -r xyz.ipa Payload/` 重新打包成 .ipa 文件  
<font color=red>注：一定要将 `Payload` 整个文件夹打包成 .ipa</font>

#### 安装
需要借助 [impactor](http://www.cydiaimpactor.com/)

1. 手机数据线连接电脑
2. 打开 `impactor`
3. 拖动 打包好的 `.ipa` 文件放到 `impactor` 上面
4. 依次输入开发者账号密码 

### 附加到 .ipa
1. 新建 iOS 项目
2. 在项目中新建 `Cocoa Touch Framework`
3. 创建 `PatchLoader` class
4. 在 `.m` 文件中添加如下代码实例
5. 确保新建的 frame 框架链接到工程
6. 运行项目后找到 `PatchPGO.dylib` 文件保存到桌面新建文件夹中
7. 下载 [https://github.com/depoon/iOSDylibInjectionDemo/blob/master/patchapp.sh](https://github.com/depoon/iOSDylibInjectionDemo/blob/master/patchapp.sh) 到文件夹
8.  `cd ~/新建文件夹`
9. 执行命令 `sh ./patchapp.sh xyz.ipa ./DYLIBS`

最后执行<font color=red>安装</font>操作

代码实例：

```
@implementation PatchLoader

static void __attribute__((constructor)) initialize(void)
{
    NSLog(@"==== Code Injection in Action====");
    /*
      Place your code injection codes here
    */
}
@end
```
参考链接:  
[https://medium.com/@kennethpoon/how-to-perform-ios-code-injection-on-ipa-files-1ba91d9438db#.ke1x2jxxv](https://medium.com/@kennethpoon/how-to-perform-ios-code-injection-on-ipa-files-1ba91d9438db#.ke1x2jxxv)

### <font color=red> static void \_\_attribute__((constructor)) initialize(void) </font>
<font color=blue> GCC </font> 
`__attribute__((constructor))`构造函数，在 `main()` 方法前执行；与之相对应的 `__attribute__((destructor)) `析构函数，在 `main()` 方法后执行

更多参考链接:  
1. [http://stackoverflow.com/questions/8433484/c-static-initialization-vs-attribute-constructor](http://stackoverflow.com/questions/8433484/c-static-initialization-vs-attribute-constructor)  
2. [http://gcc.gnu.org/onlinedocs/gcc/Function-Attributes.html](http://gcc.gnu.org/onlinedocs/gcc/Function-Attributes.html)  
3. [http://www.cnblogs.com/astwish/p/3460618.html](http://www.cnblogs.com/astwish/p/3460618.html)  
4. [http://www.cplusplus.com/forum/general/62588/](http://www.cplusplus.com/forum/general/62588/)
