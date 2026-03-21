```bash
whatweb --color=never --no-errors -a 3 -v http://10.129.7.69:80 2>&1
```

[/home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/tcp80/tcp_80_http_whatweb.txt](file:///home/rf2i/Documents/HTB/Machines/WingData/results/10.129.7.69/scans/tcp80/tcp_80_http_whatweb.txt):

```
WhatWeb report for http://10.129.7.69:80
Status    : 301 Moved Permanently
Title     : 301 Moved Permanently
IP        : 10.129.7.69
Country   : RESERVED, ZZ

Summary   : Apache[2.4.66], HTTPServer[Debian Linux][Apache/2.4.66 (Debian)], RedirectLocation[http://wingdata.htb/]

Detected Plugins:
[ Apache ]
	The Apache HTTP Server Project is an effort to develop and
	maintain an open-source HTTP server for modern operating
	systems including UNIX and Windows NT. The goal of this
	project is to provide a secure, efficient and extensible
	server that provides HTTP services in sync with the current
	HTTP standards.

	Version      : 2.4.66 (from HTTP Server Header)
	Google Dorks: (3)
	Website     : http://httpd.apache.org/

[ HTTPServer ]
	HTTP server header string. This plugin also attempts to
	identify the operating system from the server header.

	OS           : Debian Linux
	String       : Apache/2.4.66 (Debian) (from server string)

[ RedirectLocation ]
	HTTP Server string location. used with http-status 301 and
	302

	String       : http://wingdata.htb/ (from location)

HTTP Headers:
	HTTP/1.1 301 Moved Permanently
	Date: Fri, 20 Mar 2026 19:49:54 GMT
	Server: Apache/2.4.66 (Debian)
	Location: http://wingdata.htb/
	Content-Length: 345
	Connection: close
	Content-Type: text/html; charset=iso-8859-1

WhatWeb report for http://wingdata.htb/
Status    : 200 OK
Title     : WingData Solutions
IP        : 10.129.7.69
Country   : RESERVED, ZZ

Summary   : Apache[2.4.66], HTML5, HTTPServer[Debian Linux][Apache/2.4.66 (Debian)], JQuery, Script

Detected Plugins:
[ Apache ]
	The Apache HTTP Server Project is an effort to develop and
	maintain an open-source HTTP server for modern operating
	systems including UNIX and Windows NT. The goal of this
	project is to provide a secure, efficient and extensible
	server that provides HTTP services in sync with the current
	HTTP standards.

	Version      : 2.4.66 (from HTTP Server Header)
	Google Dorks: (3)
	Website     : http://httpd.apache.org/

[ HTML5 ]
	HTML version 5, detected by the doctype declaration


[ HTTPServer ]
	HTTP server header string. This plugin also attempts to
	identify the operating system from the server header.

	OS           : Debian Linux
	String       : Apache/2.4.66 (Debian) (from server string)

[ JQuery ]
	A fast, concise, JavaScript that simplifies how to traverse
	HTML documents, handle events, perform animations, and add
	AJAX.

	Website     : http://jquery.com/

[ Script ]
	This plugin detects instances of script HTML elements and
	returns the script language/type.


HTTP Headers:
	HTTP/1.1 200 OK
	Date: Fri, 20 Mar 2026 19:49:59 GMT
	Server: Apache/2.4.66 (Debian)
	Last-Modified: Sun, 02 Nov 2025 22:38:13 GMT
	ETag: "30cc-642a4410baebe-gzip"
	Accept-Ranges: bytes
	Vary: Accept-Encoding
	Content-Encoding: gzip
	Content-Length: 3764
	Connection: close
	Content-Type: text/html



```
