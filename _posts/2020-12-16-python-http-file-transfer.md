---
layout: post
title: "Easy way to Move Files with Python"
---

Short post but a fast way to move files from one computer
to another in your network is with Python.
Requires the host's (the one with the file you want) local IP address,
and access to the commandline. Note: both devices have to be accessible,
meaning same VLAN/subnet/etc. Can always test by pinging the address first.


On the host's computer, run these commands on the terminal in the directory
with the file.

```python
# python2
python -m SimpleHTTPServer 8550

# python3
python3 -m http.server 8550
```

Here, we're telling Python to import the HTTP server module with the `-m` flag.
If you don't include the port (8550 in our case), it'll default to port `8000`.
Python creates and hosts a server that can be accessed via the IP address, e.g. 192.168.1.10.

On your computer (or any device on the same network), you can now open the browser
and navigate to `http://192.168.1.10:8550` and download that file. Once you're finished,
kill the server on the host. Do note that any device can browse your files while the server is
running, so if you don't want to let everyone see what's on your computer, stick with a USB stick.
Otherwise, if it's your own home network that you feel secure about, this is one of the fastest
ways of transferring files.
