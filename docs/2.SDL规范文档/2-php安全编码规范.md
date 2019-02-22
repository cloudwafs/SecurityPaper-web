
# 02.php安全编码规范

作者：tinker

协作：

注解：请根据业务安全等级需要自行协调以下建议。

---

## 1.配置
php.ini基本安全配置

---

### 1.1应启用“cgi.force_redirect”
cgi.force_redirect在php.ini中修改，默认是开启的，它可以防止当PHP运行的CGI脚本未经验证的访问。在IIS，OmniHTTPD和Xitami上是禁用的，但在所有其他情况下它应该打开。
```php
; php.ini
cgi.force_redirect=1 ; 
```

---

### 1.2应禁用“enable_dl”
该指令仅对Apache模块版本的PHP有效。你可以针对每个虚拟机或每个目录开启或关闭dl()动态加载PHP模块。关闭动态加载的主要原因是为了安全。通过动态加载，有可能忽略所有open_basedir限制。默认允许动态加载，除了使用安全模式。在安全模式，总是无法使用dl()。
```php
; php.ini
enable_dl=0 ; 
```

---

### 1.3应禁用“file_uploads”
file_uploads默认是开启的，允许将文件上传到您的站点。因为来自陌生人的文件本质上是不可信甚至危险的，除非您的网站绝对需要，否则应禁用此功能。如果开启请进行相应的限制，参考upload_max_filesize, upload_tmp_dir,和post_max_size。 
```php
; php.ini
file_uploads = 0 ; 
```

---

### 1.4通过“open_basedir”限制文件访问权限
open_basedir默认是打开所有文件，它将 PHP 所能打开的文件限制在指定的目录树，包括文件本身。本指令不受安全模式打开或者关闭的影响。当一个脚本试图用例如 fopen() ,include或者 gzopen() 打开一个文件时，该文件的位置将被检查。当文件在指定的目录树之外时 PHP 将拒绝打开它。所有的符号连接都会被解析，所以不可能通过符号连接来避开此限制。

open_basedir应该配置一个目录，然后可以递归访问。但是，应该避免使用. （当前目录）作为open_basedir值，因为它在脚本执行期间动态解析特殊值 . 指明脚本的工作目录将被作为基准目录，但这有些危险，因为脚本的工作目录可以轻易被 chdir() 而改变。

在 httpd.conf 文件中，open_basedir 可以像其它任何配置选项一样用“php_admin_value open_basedir none”的方法关闭（例如某些虚拟主机中）。在 Windows 中，用分号分隔目录。在任何其它系统中用冒号分隔目录。作为 Apache 模块时，父目录中的 open_basedir 路径自动被继承。

用 open_basedir 指定的限制实际上是前缀，不是目录名。也就是说“open_basedir = /dir/incl”也会允许访问“/dir/include”和“/dir/incls”，如果它们存在的话。如果要将访问限制在仅为指定的目录，用斜线结束路径名。例如：“open_basedir = /dir/incl/”。 
```php
; php.ini
open_basedir="${USER}/scripts/data" ; 
```

---

### 1.5应禁用“session.use_trans_sid”
默认为 0（禁用）。当禁用cookie时，如果它开启，PHP会自动将用户的会话ID附加到URL。基于 URL 的会话管理比基于 cookie 的会话管理有更多安全风险，从表面上看，这似乎是让那些禁用cookie的用户正常使用您的网站的好方法。实际上，它使那些用户容易被任何人劫持他们的会话。例如用户有可能通过 email 将一个包含有效的会话 ID 的 URL 发给他的朋友，或者用户总是有可能在收藏夹中存有一个包含会话 ID 的 URL 来以同样的会话 ID 去访问站点。也可以从浏览器历史记录和服务器日志中检索URL获取会话ID。
```php
; php.ini
session.use_trans_sid = 0 ; 
```

---

### 1.6会话管理cookie不能是持久的
没有固定生命周期或到期日期的Cookie被称为非持久性或“会话”cookie，这意味着它们只会持续与浏览器会话一样长，并且在浏览器关闭时会消失。具有到期日期的Cookie叫做“持久性”Cookie，他们将被存储/保留到这些生存日期。

管理网站上的登录会话应用非持久性cookie。要使cookie非持久化，只需省略该 expires属性即可。也可以使用session.cookie_lifetime实现。

---

### 1.7应禁用"allow_url_fopen"和"allow_url_include"
allow_url_fopen和allow_url_include默认是开启的，他们允许代码从URL中读入脚本。从站点外部吸入可执行代码的能力，加上不完美的输入清理可能会使站点裸露给攻击者。即使该站点的输入过滤在今天是完美的，但不能保证以后也是。
```php
; php.ini
allow_url_fopen = 0
allow_url_include = 0
```

## 2.编码
php安全编码建议

---

### 2.1慎用sleep()函数
sleep()有时用于通过限制响应率来防止拒绝服务（DoS）攻击。但是因为它占用了一个线程，每个请求需要更长的时间来服务，这会使应用程序更容易受到DoS攻击，而不是减少风险。
```php
if (is_bad_ip($requester)) {
  sleep(5);  // 不合规的用法
}
```

---

### 2.2禁止代码动态注入和执行
eval()函数是一种在运行时运行任意代码的方法。
函数eval()语言结构是非常危险的，因为它允许执行任意 PHP 代码。因此不鼓励使用它。如果您仔细的确认过，除了使用此结构以外别无方法,请多加注意，不要允许传入任何由用户提供的、未经完整验证过的数据。 
```php
eval($code_to_be_dynamically_executed)  // 不合规的用法
```

---

### 2.3禁止凭据硬编码
因为从编译的应用程序中提取字符串很容易，所以永远不应对凭证进行硬编码。对于分发的应用程序尤其如此。
凭据应存储在受强保护的加密配置文件或数据库中的代码之外。
```php
// 合规的用法
$uname = getEncryptedUser();
$password = getEncryptedPass();
connect($uname, $password); 
```
```php
// 不合规的用法
$uname = "steve";
$password = "blue";
connect($uname, $password);
```

---
### 2.4禁止危险函数
有时为了安全，我们不希望执行包括system()等在那的能够执行命令的php函数，或者能够查看php（）信息的
　　phpinfo()等函数，那么我们就可以禁止比如：
　　disable_functions = system,passthru,exec,shell_exec,popen,phpinfo
　　如果你要禁止任何文件和目录的操作，那么可以关闭很多文件操作
　　disable_functions = chdir,chroot,dir,getcwd,opendir,readdir,scandir,fopen,unlink,delete,copy,mkdir, 　　rmdir,rename,file,file_get_contents,fputs,fwrite,chgrp,chmod,chown
　　这里只是列了部分不叫常用的文件处理函数，可以把上面执行命令函数和这个函数结合，
　　就能够抵制大部分的phpshell了。
  
  
  
