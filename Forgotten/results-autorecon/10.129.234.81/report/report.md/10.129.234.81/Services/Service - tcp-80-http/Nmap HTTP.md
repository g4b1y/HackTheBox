```bash
nmap -vv --reason -Pn -T4 -sV -p 80 --script="banner,(http* or ssl*) and not (brute or broadcast or dos or external or http-slowloris* or fuzzer)" -oN "/home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/tcp80/tcp_80_http_nmap.txt" -oX "/home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/tcp80/xml/tcp_80_http_nmap.xml" 10.129.234.81
```

[/home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/tcp80/tcp_80_http_nmap.txt](file:///home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/tcp80/tcp_80_http_nmap.txt):

```
# Nmap 7.98 scan initiated Wed Mar 18 23:29:26 2026 as: /usr/lib/nmap/nmap -vv --reason -Pn -T4 -sV -p 80 "--script=banner,(http* or ssl*) and not (brute or broadcast or dos or external or http-slowloris* or fuzzer)" -oN /home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/tcp80/tcp_80_http_nmap.txt -oX /home/rf2i/Documents/HTB/Machines/Forgotten/results/10.129.234.81/scans/tcp80/xml/tcp_80_http_nmap.xml 10.129.234.81
Nmap scan report for forgotten.htb (10.129.234.81)
Host is up, received user-set (0.027s latency).
Scanned at 2026-03-18 23:29:26 CET for 20s

Bug in http-security-headers: no string output.
PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.56
|_http-referer-checker: Couldn't find any cross-domain scripts.
| http-headers: 
|   Date: Wed, 18 Mar 2026 22:29:36 GMT
|   Server: Apache/2.4.56 (Debian)
|   Content-Length: 278
|   Connection: close
|   Content-Type: text/html; charset=iso-8859-1
|   
|_  (Request type: GET)
|_http-jsonp-detection: Couldn't find any JSONP endpoints.
|_http-comments-displayer: Couldn't find any comments.
|_http-wordpress-users: [Error] Wordpress installation was not found. We couldn't find wp-login.php
|_http-mobileversion-checker: No mobile version detected.
|_http-wordpress-enum: Nothing found amongst the top 100 resources,use --script-args search-limit=<number|all> for deeper analysis)
| http-vhosts: 
|_128 names had status 403
|_http-malware-host: Host appears to be clean
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-feed: Couldn't find any feeds.
|_http-litespeed-sourcecode-download: Request with null byte did not work. This web server might not be vulnerable
|_http-server-header: Apache/2.4.56 (Debian)
| http-useragent-tester: 
|   Status for browser useragent: 403
|   Allowed User Agents: 
|     Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)
|     libwww
|     lwp-trivial
|     libcurl-agent/1.0
|     PHP/
|     Python-urllib/2.5
|     GT::WWW
|     Snoopy
|     MFC_Tear_Sample
|     HTTP::Lite
|     PHPCrawl
|     URI::Fetch
|     Zend_Http_Client
|     http client
|     PECL::HTTP
|     Wget/1.13.4 (linux-gnu)
|_    WWW-Mechanize/1.34
|_http-devframework: Couldn't determine the underlying framework or CMS. Try increasing 'httpspider.maxpagecount' value to spider more pages.
| http-errors: 
| Spidering limited to: maxpagecount=40; withinhost=forgotten.htb
|   Found the following error pages: 
|   
|   Error Code: 403
|_  	http://forgotten.htb:80/
|_http-chrono: Request times for /; avg: 171.26ms; min: 160.84ms; max: 182.31ms
|_http-title: 403 Forbidden
| http-sitemap-generator: 
|   Directory structure:
|   Longest directory structure:
|     Depth: 0
|     Dir: /
|   Total files found (by extension):
|_    
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-fetch: Please enter the complete path of the directory to save data in.
|_http-date: Wed, 18 Mar 2026 22:29:32 GMT; -1s from local time.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-drupal-enum: Nothing found amongst the top 100 resources,use --script-args number=<number|all> for deeper analysis)
Service Info: Host: 172.17.0.2

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Mar 18 23:29:46 2026 -- 1 IP address (1 host up) scanned in 20.53 seconds

```
