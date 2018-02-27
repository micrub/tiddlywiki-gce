# tiddlywiki

Dockerised TiddlyWiki 5 server inside Google Compute Engine.

***This is very much a work in progress still. I am simply commiting changes as I
am developing and testing the code and doodads.***

Deploy directly to GCE instance:

```bash
make deploy EMAIL=jdoe@mywiki.com DOMAIN=mywiki.com TW_USERNAME=jdoe TW_PASSWORD=password1234
```

Test locally using Docker Compose:

```bash
make test EMAIL=jdoe@mywiki.com DOMAIN=mywiki.com TW_USERNAME=jdoe TW_PASSWORD=password1234
```

The following TCP ports should be exposed for your web browser:

| Port    | Example URL            | Permissions | Description |
| ------- | ---------------------- | ----------- | ----------- |
| tcp/80  | http://mywiki.com      | None        | HTTP automatically redirects to HTTPS |
| tcp/443 | https://mywiki.com     | Read-only   | HTTPS TiddlyWiki (PUT and DELETE writes are silently ignored, responding with HTTP 405 response codes) |
| tcp/444 | https://mywiki.com:444 | Read-write  | HTTPS TiddlyWiki (password protected with basic digest authentication) |

See https://nicolaw.uk/gcloud and https://nicolaw.uk/gcp for tangentally related
notes on Google Cloud Platform.

## Design Overview

```
                                                                      |\_/|
                                                                     / @ @ \
                                                 End-user browsing  ( ^ º ^ )
                                                 https://mywiki.com  `>>x<<´
                                                                     /  O  \

                                                                        +
+--[Google Compute Engine]----------------------------------------------|------------------------------------+
|                                                                       |                                    |
|  +--[Container-Optimized OS VM Instance]------------------------------|---------------------------------+  |
|  |                                                                    |                                 |  |
|  |  +--[Docker]-----------+  +--[Docker]-----------+  +--[Docker]-----|-----+  +--[Docker]-----------+  |  |
|  |  |                     |  |                     |  |               |     |  |                     |  |  |
|  |  | Scheduled automation|  |  TiddlyWiki NodeJS  |  |  Apache       v     |  |  LetsEncrypt SSL    |  |  |
|  |  | tasks push backups  |  |  exposes tcp/8080   |  |  exposes            |  |  certificate        |  |  |
|  |  | to Git, & generate  |  |  to Apache.         |  |  tcp/80,443,444     |  |  generation         |  |  |
|  |  | static wiki page    |  |                     |  |  to the Internet.   |  |     automation and  |  |  |
|  |  | renderings.      <-------+                <-------+                <-------+   renewal.        |  |  |
|  |  |                 +   |  |                     |  |                     |  |                     |  |  |
|  |  +-----------------|---+  +---------------------+  +---------------------+  +---------------------+  |  |
|  |                    |                                                                                 |  |
|  +--------------------|---------------------------------------------------------------------------------+  |
|                       |                                                                                    |
+-----------------------|------------------------------------------------------------------------------------+
                        |
+--[Git Repository]-----|------+
|                       |      |
|  Upstream Git repo    v      |
|  provides versioned          |
|  backup store of TiddlyWiki  |
|  pages and static content.   |
|                              |
+------------------------------+
```

![Terminal screen capture exmaple of make test](https://i.imgur.com/4uXLEkR.gif)

## TODO

* Setup pull (and push) of Tiddlers to and from GitHub repository.
* Make LetsEncrypt functionality a little more robust.
* Finish deployment to GCE functionality.
* Make configuration tunable via cloud metadata (use
  https://cloud.google.com/compute/docs/storing-retrieving-metadata).
* Possibly use Terraform to orchestrate instance and Docker registry image
  deployment to cloud platform.
* Replace Apache with Nginx for better protection against slowloris?
* Combine tiddlywiki and automation images, but retain as two seperate runtime
  containers.

## License

MIT License

Copyright (c) 2018 Nicola Worthington

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
