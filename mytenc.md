## Qz心得
### cgi或者静态页面404
1. 没有配置static_module. 加上<ModName="abc_xyz_static"runtype="static"/>
2. 检查hdocs的目录是否配置正确
3. 空域名 virtuelHost="" 放到最后 否则静态页面无法访问
4. /abc/xyz/cgi-bin/abc.fcg : CgiHome="/abc/xyz" ModName="cgi-bin/abc.fcg" . url : http://xxx.qq.com/cgi-bin/abc.fcg
5. 配置了域名不能直接用ip 访问 需要配置 VirtuelHost="xxx.qq.com:8081" 要ip访问加上VirtuelHost="127.0.0.1:8081"

### 返回500
1. cgi超时 可能proxy日志得出
2. cgi coredump
3. cgi returntype 不对 比如fastcgi 设置为 cgi
4. cgi没有正确启动

userName 是linux 用户名， 一定要设置为低权限用户，如果用root启动可以设置为其他任意非root用户，如果用非root用户必须启动时设置自己的用户名。