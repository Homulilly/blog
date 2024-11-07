---
title: 安装 Logwatch 自动获取 VPS 日志报告
date: 2016-09-27 19:07:47
categories:
 - Note
tags:
 - Ubuntu
 - Logwatch
 - Postfix
---
服务器以及软件运行时肯定会产生很多的日志，手动看肯定不是个法，可以安装日志分析软件来进行处理。
Logwatch 是一款开源的日志分析器，主要用于生成日志摘要，而且可以配合 postfix 通过邮件发送报告。  

虽然 Logwatch 并不是用于对日志进行实时地分析和监控，但是对于个人用于通过获取定期的日志分析报告来关注服务器的运行状况是很适用的。并且安装与使用也较为方便。
<!--more-->

### 安装 Logwatch
Ubuntu / Debian 安装：
```
sudo apt-get install logwatch
```
安装 Logwatch 后会自动一起安装 Postfix 用于发送邮件，如果将日志通过邮件发送，需要提前配置好 Postfix。

### 配置与运行 Logwatch  
Logwatch 运行的选项会依次读取，优先级高的中所存在的选项会覆盖低优先级中的内容
1. 运行时命令行直接指定的内容
2. 配置文件 `/etc/logwatch/conf/logwatch.conf` (需要自行创建)
3. 配置文件 `/usr/share/logwatch/default.conf/logwatch.conf`

创建 `/etc/logwatch/conf/logwatch.conf`， 该文件中的内容会覆盖 `/usr/share/logwatch/default.conf/logwatch.conf` 中的默认选项。  

`logwatch.conf` 示例：
```bash
MailTo = email_address@domain.com
MailFrom = logwatch
Detail = 10
Range = Yesterday
Service = http
Service = sshd
Format = html
Output = mail
```

- `MailTo`  
  表示报告的收件人的邮件地址

- `MailFrom`   
  报告的发件人的邮件地址，填写用户名则发件域名会从 postfix 中读取，或是直接填写完整的邮件地址

- `Detail`  
  指报告的详细程度    
  可以填写 High, Med, Low 或是分别起对应的数字 10, 5， 0

- `Range`  
  表示分析日志的时间范围  
  可填 Yesterday, Today, All, 选择 All 时需要追加填写 `Archive = yes` 表示分析已归档的日志内容。

- `Service`  
  需要分析的内容  
  可以填写 All 或是特定 service_name， 选择 All 时，可以追加填写 "-service_name" 表示排除某一服务。如需单独指定时需要注释掉 `/usr/share/logwatch/default.conf/logwatch.conf` 中的 `Service = All`  
  默认所有支持的服务可以通过 `ls /usr/share/logwatch/scripts/services` 查看。

- `Format`    
  报告输出的格式  
  `html` 或 `txt`

- `Output`  
  报告输出的方式  
  `file` `mail` 或是 `stdout`
  选择 `file` 时可以追加 `Filename = [path to file]` 表示存储文件的位置。
  比如可以先使用命令行测试生成文件 `logwatch --ouput file --filename $HOME/output.html`，根据我执行时的情况，似乎这里指定 filename 的路径时， Logwatch 会忽视大小写，所以最好不要使用带有大写字母的路径。

### 设置自动运行 Logwatch  
默认在 `/etc/cron.daily` 会创建一个 `00logwatch` 文件，每日运行一次 Logwatch ，查看 `cron.daily` 运行的时间可以通过 `grep run-parts /etc/crontab` 查看：
```
> grep run-parts /etc/crontab
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
```

可以修改对应的时间或是删除 Logwatch 的文件，通过 `crobtab -e` 设置为自行指定的时间：
```
#每日0点5分运行
5 0 * * * /usr/sbin/logwatch
```

### 配置 Postfix 使用 SparkPost 发送邮件
安装 Logwatch 过程中已经安装了 postfix ，Logwatch 会使用 postfix 的 sendmail 发送邮件，无需安装其他软件。

配置 Postfix：  
编辑 Postfix 配置文件 `/etc/postfix/main.cf `  
- 设置发送域名  
  ```
  myhostname = domain.name
  myorigin = $myhostname
  #or myorigin=/etc/mailname
  ```
- 设置使用 SMTP 通过 SparkPost 发送邮件  
  via [SparkPost](https://developers.sparkpost.com/)  
  去 SparkPost 创建一个有 SMTP 发送权限的 api key ，然后将下面内容加入 配置文件之中
  ```
  smtp_sasl_auth_enable = yes
  smtp_sasl_password_maps = static:SMTP_Injection:<your API key>
  relayhost = [smtp.sparkpostmail.com]:587
  smtp_sasl_security_options = noanonymous
  smtp_tls_security_level = encrypt
  header_size_limit = 4096000
  ```
- 设置 Postfix 仅接受本地请求   
  架设了邮件发送服务器就要注意安全，设为仅接受本地网络请求较为保险。
  指定 `inet_interfaces` 为 `loopback-only`：
  ```
  inet_interfaces = loopback-only
  ```

- 重启并检查 Postfix 服务  
  ```
  sudo service postfix restart
  ```
  使用 `postconf -n` 查看当前 postfix 的设置  
  使用 `netstat -tunlp` 查看监听的端口为 `127.0.0.1:25` 而非 `0.0.0.0:25`  
  使用 `telnet 127.0.0.1 25` 可以查看端口打开情况

- 测试邮件发送  
  使用 mailutils 中的 mail 测试命令，使用 mail 命令时发送邮件的域名服务器会使用 postfix 中指定的域名，可以手动通过 `-a F username@domain.com` 指定完整的发件人邮件地址 ：
  ```
  echo "Mail Content" | mail -a F username@domain.com -s "Mail Subject" address@domain.com && tail -f /var/log/mail.log
  ```

- 查看 Log  
  Postfix 的 log 位于 `/var/log/mail/log`
  ```
  less /var/log/mail/log
  ```

参考链接：
- [Linux 系统中使用 logwatch 监控日志文件](https://linux.cn/article-4490-1.html)
- [在Linux上使用logwatch分析监控日志文件](https://segmentfault.com/a/1190000002537665)
- [Logwatch简单配置教程](https://www.mawenbao.com/note/logwatch.html)
- [安装和配置 Postfix](http://chloerei.com/2015/04/22/install-and-configure-postfix/)
- [使用 Postfix 设置只发送邮件的邮件服务](http://www.chrisyue.com/config-postfix-as-a-send-only-mail-server.html)
- [POSTFIX MAIIN.CF几个重要参数中文解释](http://bbs.51cto.com/thread-589893-1.html)
