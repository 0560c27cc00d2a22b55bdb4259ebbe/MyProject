
iOS 10.2 非完美越狱，越狱失效后需重在pp助手中再次一键越狱，随后删掉手机上老版本的越狱助手，并启动新安装的越狱助手重新越狱。 不要在手机上安安装OpenSSH-


#### ssh 失败
1. 在 `Cydia` 中安装 `Filza`
2. 启动 `Filza`
3. 根据路径 `var/containers/Bundle/Application/越狱助手/yalu102.app/` 找到  `dropbear.plist`
4. 将 `dropbear.plist` 中 `Root/ProgramArguements/item4` 值修改为 `22`
5. 重启

#### scp 失败
将相同型号的手机 `/usr/bin/scp` 路径下的 `scp` 拷贝到 `10.2` 版本的手机中。

```
$ ssh root@192.168.101.168
# cd /usr/bin
# ldid -S scp
# chmod 777 scp
# mv scp /usr/bin/scp
```
重启搞定！

#### `lldb + debugserver` 与其他版本相同，一些查看 `log` 的插件不能正常使用了，在 `Xcode` 终也无法正常查看（会不停的打印系统信息）。
