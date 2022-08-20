# HTTP headers in nginx-ingress

## X-Frame-Options:
Allowed to browser render a page in a `<frame>`, `<iframe>`, `<embed>`, `<object>`. Using `X-Frame-Options` response header to avoid click-jacking attack.  
There are three directives for `X-Frame-Options`:
- DENY: The page cannot be displayed in a frame.
- SAMEORIGIN: The page can only displayed in a frame on the same origin as the page itself.
Add X-Frame-options on nginx ingress:  
```
nginx.ingress.kubernetes.io/configuration-snippet: |   
  add_header X-Frame-Options "sameorigin";
```

## X-XSS-Protection:
It tells the browser to stop pages from loading when detect XSS attacks.
There are four directives for `X-XSS-Protection`:
- X-XSS-Protection: 0; - Disable XSS filtering.
- X-XSS-Protection: 1; - Enables XSS filtering. 
- X-XSS-Protection: 1; mode=block - Enables XSS filtering. Rather than sanitizing the page, the browser will prevent rendering of the page if an attack is detected.
- X-XSS-Protection: 1; report= reporting-uri; - Enables XSS filtering. This uses the functionality of the CSP report-uri directive to send a report.


## Content-Security-Policy:
CSP is an added layer of security that helps to detect and mitigate certain types of attacks. 
To enable CSP, just to configuration server CSP headers.
- script-src: assign source allow to load js script.
- style-src: assign source allow to load css.
- image-src: assign source allow to load image.
- font-src: assign source allow to load font.
- frame-src: assign source allow to load frame.

The values of CSP directive:
- `*` : allow all connects.
- self:  allow content from a by self domain.
- none: block all.
- http://www.abc.zyx: allow source for domain allocate, is different for abc.xyz
- abc.zyx: allow source for domain allocate, not allow http://www.abc.xyz, abc.def.xyz, cdn.abc.xyz, ...
- *.abc.xyz: allow load for subdomain.
- https: allow load website have https.

## Strict-Transport-Security:
HTTP STS header tells the browsers that the site should only be accessed using HTTPS.  
- Strict-Transport-Security: max-age=`<expire-time>`;
- Strict-Transport-Security: max-age=`<expire-time>`; includeSubDomains :  This is a more secure option but will block access to certain pages that can only be served over HTTP
- Strict-Transport-Security: max-age=`<expire-time>`; preload : Permits browsers to automatically preload HSTS configuration. Prevents an attacker from downgrading a first request from HTTPS to HTTP. 
- Strict-Transport-Security: max-age=`<expire-time>`; includeSubDomains; preload

## X-Content-Type-Options:
The X-Content-Type-Options response HTTP header is a marker used by the server to indicate that the MIME types advertised in the Content-Type headers should be followed and not be changed. The header allows you to avoid MIME type sniffing by saying that the MIME types are deliberately configured.
- X-Content-Type-Options: nosniff

## Server Leaks Information via 'X-Powered-By' HTTP Response Header
Ensure that your web server, application server, load balancer, etc. is configured to suppress 'X-Powered-By' headers.
```
nginx.ingress.kubernetes.io/configuration-snippet: |
  more_clear_headers "Server";
```