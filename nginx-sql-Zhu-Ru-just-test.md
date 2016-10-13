代码不可能是全完美的，动态网页在实用中难免会遇到sql注入的攻击。
而通过nginx的配置过滤，可以很好的避免被攻击的可能。
SQL注入攻击一般问号后面的请求参数，在nginx里用$query_string表示 。

##### $request_uri
This variable is equal to the *original* request URI as received from the client including the args.
It cannot be modified. Look at $uri for the post-rewrite/altered URI. Does not include host name.
Example: "/foo/bar.php?arg=baz"
这个变量等于从客户端发送来的原生请求URI，包括参数。它不可以进行修改。$uri变量反映的是重写后/改变的URI。
不包括主机名张小三资源网。例如："/foo/bar.php?arg=baz"
##### $uri
This variable is the current request URI, without any arguments (see $args for those).
This variable will reflect any modifications done so far by internal redirects or the index module.
Note this may be different from $request_uri, as $request_uri is what was originally sent by the browser before any such modifications.
Does not include the protocol or host name. Example: /foo/bar.html
这个变量指当前的请求URI，不包括任何参数(见$args)。这个变量反映任何内部重定向或index模块所做的修改。
注意，这和$request_uri不同，因$request_uri是浏览器发起的不做任何修改的原生URI。
不包括协议及主机名。例如张小三资源："/foo/bar.html"
##### $document_uri
The same as $uri.
同$uri.

#### 一、特殊字符过滤

例如URL /plus/list.php?tid=19&mid=22' ，后面带的单引号为非法的注入常用字符。而想避免这类攻击，可以通过下面的判断进行过滤。

```sql
if ( $query_string ~* ".*[;'<>].*" ) {
  return 404;
}
```

#### 二、sql语句过滤
##### 内部
```sql
if ($uri ~* (.*)(insert|select|delete|update|count|master|truncate|declare|exec|\*|%|\')(.*)$ ) { return 403; }
```
##### 外部
```sql
if ($request_uri ~* "(cost\()|(concat\()") { return 403; }
if ($request_uri ~* "[+|(%20)]union[+|(%20)]") { return 403; }
if ($request_uri ~* "[+|(%20)]and[+|(%20)]") { return 403; }
if ($request_uri ~* "[+|(%20)]select[+|(%20)]") { return 403; }
if ($request_uri ~* "[+|(%20)]or[+|(%20)]") { return 403; }
if ($request_uri ~* "[+|(%20)]delete[+|(%20)]") { return 403; }
if ($request_uri ~* "[+|(%20)]update[+|(%20)]") { return 403; }
if ($request_uri ~* "[+|(%20)]insert[+|(%20)]") { return 403; }
```

#### 三、文件注入禁止

```sql
if ($query_string ~ "[a-zA-Z0-9_]=http://") { return 403; }
if ($query_string ~ "[a-zA-Z0-9_]=(\.\.//?)+") { return 403; }
if ($query_string ~ "[a-zA-Z0-9_]=/([a-z0-9_.]//?)+") { return 403; }
```

#### 四、溢出攻击过滤

```sql
if ($query_string ~ "(<|%3C).*script.*(>|%3E)") { return 403; }
if ($query_string ~ "GLOBALS(=|\[|\%[0-9A-Z]{0,2})") { return 403; }
if ($query_string ~ "_REQUEST(=|\[|\%[0-9A-Z]{0,2})") { return 403; }
if ($query_string ~ "proc/self/environ") { return 403; }
if ($query_string ~ "mosConfig_[a-zA-Z_]{1,21}(=|\%3D)") { return 403; }
if ($query_string ~ "base64_(en|de)code\(.*\)") { return 403; }
```

#### 五、spam字段过滤

```sql
set $block_spam 0;
if ($query_string ~ "\b(ultram|unicauca|valium|viagra|vicodin|xanax|ypxaieo)\b") { return 403; }
if ($query_string ~ "\b(erections|hoodia|huronriveracres|impotence|levitra|libido)\b") { return 403; }
if ($query_string ~ "\b(ambien|blue\spill|cialis|cocaine|ejaculation|erectile)\b") { return 403; }
if ($query_string ~ "\b(lipitor|phentermin|pro[sz]ac|sandyauer|tramadol|troyhamby)\b") { return 403; }
```

#### 六、user-agents头过滤
```sql
if ($http_user_agent ~ ApacheBench|WebBench|Jmeter|JoeDog|Havij|GetRight|TurnitinBot|GrabNet|masscan|mail2000|github|wget|curl) { return 403; }
if ($http_user_agent ~ "Go-Ahead-Got-It") { return 403; }
if ($http_user_agent ~ "GetWeb!") { return 403; }
if ($http_user_agent ~ "Go!Zilla") { return 403; }
if ($http_user_agent ~ "Download Demon") { return 403; }
# Disable Akeeba Remote Control 2.5 and earlier
if ($http_user_agent ~ "Indy Library") { return 403; }
# Common bandwidth hoggers and hacking tools.
if ($http_user_agent ~ "libwww-perl") { return 403; }
if ($http_user_agent ~ "Nmap Scripting Engine") { return 403; }
if ($http_user_agent ~ "~17ce.com") { return 403; }
if ($http_user_agent ~ "WebBench*") { return 403; }
if ($http_referer ~* 17ce.com) { return 403; }
if ($http_referer ~* WebBench*") { return 403; }
```

上文列出的是普遍的方法，针对各自现网应用的不同，使用上也需要进行相应的调整。下面列出一下我现网中的防止方法。

#### 一、自动防护

```sql
if ($request_uri ~* .(htm|do)?(.*)$) { set $req $2; }
if ($req ~* "(cost\()|(concat\()") { return 503; }
if ($req ~* "union[+|(%20)]") { return 503; }
if ($req ~* "and[+|(%20)]") { return 503; }
if ($req ~* "select[+|(%20)]") { return 503; }
```

* 这里之所以使用$request_uri而未使用$query_string变量，因为通过$request_uri进行rewrite分割更精准。
* %20代表的是空格，同上文不的是，我这里把上面的空格匹配进行了取消。这样像www.361way.com/aaa.do?select * from test之样的也可以进行匹配。
* 上面的htm是伪静态，实际上同.do一样，也是动态文件。为了便于和静态文件进行区分，这里选择了htm而不是html。
* 注意，最上面的url里面的\? ，这个也分重要。如果没有的话，www.361way.com/aaa.htm select * from test不会被过滤，而www.361way.com/aaa.htm?select * from test会被过滤。如果想将前面的也过滤，只需要把\? 取消即可。

#### 二、日志获取，手动分析

具体哪些url有可能有注入漏洞而被人扫描了，可以利用下面的脚本并通过mail发送。

```shell
#!/bin/bash
cd /tmp
/bin/rm -rf nginxanalay.tar.gz
cd /logs/nginx
egrep  '(sqlmap|select|"order by")' *|egrep -v '(Googlebot|Baiduspider|Sosospider|stepselect)'|awk -F 'HTTP/1.1"' '{print $1}' > /tmp/nginxanalay.log
cd /tmp
tar czvf nginxanalay.tar.gz nginxanalay.log
/usr/bin/sendEmail -f nagios@111cn.net -t 收件人1 收件人2 -s mail.111cn.net -u 'site sql analay' -m 'this is nginxlog analay . see Annex ,That is
 may be injected into page .' -xu 用户名 -xp 密码 -a /tmp/nginxanalay.tar.gz
```
