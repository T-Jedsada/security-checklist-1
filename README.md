# Security Checklist

[SecurityChecklist.org](https://securitychecklist.org) | [Hacker News Discussion](https://news.ycombinator.com/item?id=11323849)

- [ ] Is the website only served over https?

```sh
$ curl -s -I http://example.org | grep '^HTTP'
HTTP/1.1 301 Moved Permanently
$ curl -s -I https://example.org | grep '^HTTP'
HTTP/1.1 200 OK
```

- [ ] Is the HSTS http-header set?

```sh
$ curl -s -I https://example.org | grep '^Strict'
Strict-Transport-Security: max-age=63072000; includeSubdomains; always
```

- [ ] Is the server certificate at least 4096 bits?

```sh
$ openssl s_client -showcerts -connect example.org:443 |& grep '^Server public key'
Server public key is 4096 bit
```

- [ ] Is TLS1.2 the only supported protocol?

```sh
$ curl --sslv3 https://example.org
curl: (35) Server aborted the SSL handshake
$ curl --tlsv1.0 -I https://example.org
curl: (35) Server aborted the SSL handshake
$ curl --tlsv1.1 -I https://example.org
curl: (35) Server aborted the SSL handshake
$ curl --tlsv1.2 -s -I https://example.org | grep 'HTTP'
HTTP/1.1 200 OK
```

- [ ] Do all supported symmetric ciphers use at least 256 bit keys?

```sh
$ nmap --script ssl-enum-ciphers -p 443 example.org
PORT    STATE SERVICE
443/tcp open  https
| ssl-enum-ciphers:
|   TLSv1.2:
|     ciphers:
|       TLS_DHE_RSA_WITH_AES_256_CBC_SHA - strong
|       TLS_DHE_RSA_WITH_AES_256_CBC_SHA256 - strong
|       TLS_DHE_RSA_WITH_AES_256_GCM_SHA384 - strong
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA - strong
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 - strong
|       TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 - strong
|     compressors:
|       NULL
|_  least strength: strong              
```

- [ ] Is the Diffie-Hellman prime at least 4096 bits?

```sh
$ openssl s_client -connect example.com:443 -cipher "EDH" |& grep "^Server Temp Key"
Server Temp Key: DH, 4096 bits
```

- [ ] Have you ensured that your content cannot be embedded in a frame on another website?

```sh
$ curl -s -I https://example.org | grep '^X-Frame-Options'
X-Frame-Options: SAMEORIGIN
$ curl -s -I https://example_2.org | grep '^X-Frame-Options'
X-Frame-Options: DENY # Also acceptable
```

- [ ] Have you ensured that the Internet Explorer content sniffer is disabled?

```sh
$ curl -s -I https://example.org | grep '^X-Content'
X-Content-Type-Options: nosniff
```

- [ ] Do all assets delivered via a content delivery network include subresource integrity hashes?

```html
<link
  rel="stylesheet"
  href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.2/css/bootstrap.min.css"
  integrity="sha384-y3tfxAZXuh4HwSYylfB+J125MxIs6mR5FOHamPBG064zB+AFeWH94NdvaCBm8qnd"
  crossorigin="anonymous"
>
```

- [ ] Are password entropy checks done during user sign-up, using, say AUTH_PASSWORD_VALIDATORS?

- [ ] Are you storing only the hash of your users password, and not the cleartext password, using (say) [BCrypt](https://codahale.com/how-to-safely-store-a-password/)?

- [ ] Are failed login attempts throttled and IP addresses banned after a number of unsuccessful attempts, using (say) [Rack::Attack](https://github.com/kickstarter/rack-attack)?

- [ ] Are you only allowing ssh login attempts via VPN or using fail2ban to throttle ssh login attempts?

```sh
sudo fail2ban-client status sshd
```

- [ ] Have you disabled password-based login over ssh, and only allowed key-based login?

```sh
$ cat /etc/ssh/sshd_config  | grep '^Password'
PasswordAuthentication no
```

- [ ] Do session cookies have the 'Secure' and 'HttpOnly' flag set?

```sh
$ curl -s -I example.com/url_that_sets_cookie | grep '^Set-Cookie'
Set-Cookie: ****;Path=/;Expires=Fri, 16-Mar-2018 19:18:51 GMT;Secure;HttpOnly;Priority=HIGH
```

- [ ] Do forms set a cross-site request forgery cookie?

```sh
$ curl -s -I https://example.com/url_with_form | grep '^Set-Cookie'
Set-Cookie: csrftoken=*****************; expires=Thu, 16-Mar-2017 01:26:03 GMT;Secure;HttpOnly; Max-Age=31449600; Path=/
```

- [ ] Are all user uploads validated for expected content type?

- [ ] Are the permissions of all uploaded files readonly?

- [ ] Are all form fields (with the exception of password fields) validated with a restrictive regex?

- [ ] Are there unit tests (say, using [Selenium](http://www.seleniumhq.org)) which show that one authenticated user cannot access another user's content?

- [ ] Have you made sure that database passwords, server signing keys, and hash salts are not checked into source control?

- [ ] Do you have an account recovery flow? Be aware that this can jeopardize everything on this checklist.