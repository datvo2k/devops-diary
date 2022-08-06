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
