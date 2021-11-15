title: Send HTTP requests with netcat
date: 2021-11-15 17:28:18
tags:
  - http
---
Occasionally I need to call HTTP API from a bare-bones environment, where we don't have `curl` or `wget`. Nowadays, it's so common that we run everything in the dockerized environment, and we are apt to use a slim docker image without many utility libraries.

<!-- more -->

Of course, you would say installing the `curl` is the most straightforward solution here. Still, unfortunately, we don't always have the privilege to install packages in a built container, so we need an alternative to make the HTTP request; here comes the `netcat` command, it's usually pre-installed in Linix based system out-of-box.


Before I try to use `netcat` to send the HTTP request, I need APIs to be called. I use https://github.com/namshi/mockserver to create two simple APIs(one GET and one POST)


Response definition of `mocks/foo/GET.mock`

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
   "Random": "content"
}
```


Response definition of `mocks/foo/POST.mock`

```
HTTP/1.1 200 OK
Content-Type: text/xml; charset=utf-8

{
   "Accept-Language": "en-US,en;q=0.8",
   "Host": "headers.jsontest.com",
   "Accept-Charset": "ISO-8859-1,utf-8;q=0.7,*;q=0.3",
   "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"
}
```
You can run `mockserver -p 8080 -m mocks` so you have two APIS available for testing.

## use nc to send HTTP GET request

```
printf 'GET /foo HTTP/1.0\r\nHost:localhost\r\n\r\n' | nc  -v localhost 8080
```
**Caveat**: you have to specify the `Host` as it's mandaory according to the HTTP protocol.
You will see the response is printed as the stdout.

```
Connection to localhost port 8080 [tcp/http-alt] succeeded!
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Mon, 15 Nov 2021 09:47:02 GMT
Connection: close

{
   "Random": "content"
}
```

## use nc to send HTTP POST request

POST needs more work as you have to orchestrate the body and related headers, so I ended up with a bash script.

```
#!/bin/bash
BODY="{ \"bar\" : \"Hello\", \"baz\" : \"11111\"}"

echo -ne "POST /foo HTTP/1.0\r\nHost: localhost\r\nContent-Type: application/json\r\nContent-Length: ${#BODY}\r\n\r\n${BODY}" | nc -i 3 localhost 8080
```

That's pretty much it.
