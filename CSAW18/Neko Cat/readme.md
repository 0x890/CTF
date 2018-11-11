# NekoCat WEB 500 
![NikoCat](http://web.chal.csaw.io:1003/static/favicon.ico)
## Difficulty: ★★★★★

Link: http://web.chal.csaw.io:1003

# Idea

> * Bypass CSP by Abusing XSS Filter using inline
> * Escalate user permissions
> * Abusing URL Parser to read environment variables
> * Craft malicious session lead to RCE

# Walkthrough
description provide us the source code of the challenge 
[flagon.zip](https://ctf.csaw.io/files/a75d5b38afc3d477873e8ce01c468d85/flagon.zip)

we notice that challange uses Python Flask
so let's create account and try to see what's happen there 
http://web.chal.csaw.io:1003/register

![NikoCat](https://screenshotscdn.firefoxusercontent.com/images/ae6ec98e-8b7c-4909-9fc6-a2c6fe880bf5.png)

we try to post link but we can see only verfied users can preview link 

> app.py Line:152
```python
if verified_user(session, request.session.get('username'))[0]:
    preview = get_post_preview(link)
 ```
so the idea here is to get permession of verfied user in plateform
we can trigger an XSS attack to steal cookies from admin maybe
but when we dig more in the source code we see there is CSP rules implemented
> app.py Line:51
```python
def apply_csp(f):
    @wraps(f)
    def decorated_func(*args, **kwargs):
        resp = f(*args, **kwargs)
        csp = "; ".join([
                "default-src 'self' 'unsafe-inline'",
                "style-src " + " ".join(["'self'",
                                         "*.bootstrapcdn.com",
                                         "use.fontawesome.com"]),
                "font-src " + "use.fontawesome.com",
                "script-src " + " ".join(["'unsafe-inline'",
                                          "'self'",
                                          "cdnjs.cloudflare.com",
                                          "*.bootstrapcdn.com",
                                          "code.jquery.com"]),
                "connect-src " + "*"
              ])
        resp.headers["Content-Security-Policy"] = csp

        return resp
    return decorated_func
```
