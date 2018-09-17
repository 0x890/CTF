# Difficulty: ★★☆☆☆

Sovled: 210 / 1448

Tag: SINGLE SIGN ON, oauth2, JWT token

Link: web.chal.csaw.io:9000

# Idea

> Use oauth 2.0 for authentication and request the resources
> Get jwt secret to craft a token in order to admin permissions and read the flag

# Request 

> GET / HTTP/1.1
> Host: web.chal.csaw.io:9000
# Response
```
  <h1>Welcome to our SINGLE SIGN ON PAGE WITH FULL OAUTH2.0!</h1>
  <a href="/protected">.</a>
  <!--
  Wish we had an automatic GET route for /authorize... well they'll just have to POST from their own clients I guess
  POST /oauth2/token
  POST /oauth2/authorize form-data TODO: make a form for this route
  --!>
```

![Get Request to main page](https://ucarecdn.com/9c4256f5-f20d-4bde-bfca-67ee2b6a64b1/0.png)

and
