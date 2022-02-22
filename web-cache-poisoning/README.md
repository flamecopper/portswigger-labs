<!-- omit in toc -->
# Web cache poisoning with an unkeyed header

<!-- omit in toc -->
## Table of Contents

- [Web cache poisoning with an unkeyed header](#web-cache-poisoning-with-an-unkeyed-header)
- [Web cache poisoning with an unkeyed cookie](#web-cache-poisoning-with-an-unkeyed-cookie)

## Web cache poisoning with an unkeyed header
Reference: https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-with-an-unkeyed-header

<!-- omit in toc -->
### Solution 
1. With Burp running, load the website's home page
2. In Burp, go to "Proxy" > "HTTP history" and study the requests and responses that you generated. Find the GET request for the home page and send it to Burp Repeater.
3. Add a cache-buster query parameter, such as ?cb=1234.
4. Add the ``X-Forwarded-Host`` header with an arbitrary hostname, such as ``example.com``, and send the request.
5. Observe that the X-Forwarded-Host header has been used to dynamically generate an absolute URL for importing a JavaScript file stored at ``/resources/js/tracking.js``.
6. Replay the request and observe that the response contains the header ``X-Cache: hit``. This tells us that the response came from the cache.
7. Go to the exploit server and change the file name to match the path used by the vulnerable response:
```
/resources/js/tracking.js
```
8. In the body, enter the payload alert(document.cookie) and store the exploit.
9. Open the GET request for the home page in Burp Repeater and remove the cache buster.
10. Add the following header, remembering to enter your own exploit server ID:
```
X-Forwarded-Host: your-exploit-server-id.web-security-academy.net
```
11. Send your malicious request. Keep replaying the request until you see your exploit server URL being reflected in the response and X-Cache: hit in the headers.
12. To simulate the victim, load the poisoned URL in your browser and make sure that the alert() is triggered. Note that you have to perform this test before the cache expires. The cache on this lab expires every 30 seconds.
13. If the lab is still not solved, the victim did not access the page while the cache was poisoned. Keep sending the request every few seconds to re-poison the cache until the victim is affected and the lab is solved.

## Web cache poisoning with an unkeyed cookie
Reference: https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-with-an-unkeyed-cookie

<!-- omit in toc -->
### Solution
1. With Burp running, load the website's home page.
2. In Burp, go to "Proxy" > "HTTP history" and study the requests and responses that you generated. Notice that the first response you received sets the cookie ``fehost=prod-cache-01``.
3. Reload the home page and observe that the value from the ``fehost`` cookie is reflected inside a double-quoted JavaScript object in the response.
4. Send this request to Burp Repeater and add a cache-buster query parameter.
5. Change the value of the cookie to an arbitrary string and resend the request. Confirm that this string is reflected in the response.
6. Place a suitable XSS payload in the ``fehost`` cookie, for example:
```
fehost=someString"-alert(1)-"someString
```
7. Replay the request until you see the payload in the response and ``X-Cache: hit`` in the headers.
8. Load the URL in your browser and confirm the ``alert()`` fires.
9. Go back Burp Repeater, remove the cache buster, and replay the request to keep the cache poisoned until the victim visits the site and the lab is solved.