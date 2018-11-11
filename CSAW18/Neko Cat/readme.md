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

