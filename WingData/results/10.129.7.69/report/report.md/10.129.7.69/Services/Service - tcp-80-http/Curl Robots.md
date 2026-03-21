```bash
curl -sSikf http://10.129.7.69:80/robots.txt
```

[/home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/tcp80/tcp_80_http_curl-robots.txt](file:///home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/tcp80/tcp_80_http_curl-robots.txt):

```
HTTP/1.1 301 Moved Permanently
Date: Fri, 20 Mar 2026 19:49:51 GMT
Server: Apache/2.4.66 (Debian)
Location: http://wingdata.htb/robots.txt
Content-Length: 355
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html><head>
<title>301 Moved Permanently</title>
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="http://wingdata.htb/robots.txt">here</a>.</p>
<hr>
<address>Apache/2.4.66 (Debian) Server at 10.129.7.69 Port 80</address>
</body></html>

```
