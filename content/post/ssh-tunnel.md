---
title: "Ssh Tunnel"
date: "2017-12-27T23:30:17+08:00"
lastmod: "2017-12-27T23:30:17+08:00"
draft: false
tags: ["izhouyan", "linux"]
categories: ["linux", "ssh", "tool", "network"]
author: "izhouyan"

---


There are three machines, A, B and C. A can not reach C, but A and B, B and C are connected, how to use ssh tunnel to connect A and C?

All config is set on machine B.

# B to C

create a tunnel from B to C, using local forwarding

> ssh -L 2001:localhost:9000 C -N

The –L option specifies local forwarding, in which the TCP client is on the local machine with the SSH client.
The option is followed by three values separated by colons: a local port to listen on (2001), the remote machine name or IP address (C), and the remote, target port number (9000).

# B to A

create a tunel from B to A, using remote forwarding

> ssh -R 9000:localhost:2001 A -N

The –R option specifies remote forwarding. The remote port to be forwarded (9000) is now first, followed by the machine name or IP address (localhost) and port number (2001). SSH can now forward connections from (localhost,2001) to (A,9000).

# Usage

Now, create a http server on C.

> python2 -m SimpleHTTPServer 9000

Connect from A, we can see the result.

> wget localhost:9000

We create a ssh tunnel on machine B using port 2001, to connect port 9000 on machine A and C.

That's it.
