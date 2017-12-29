---
title: "Read http server log with command tools"
date: "2017-12-29T23:21:04+08:00"
lastmod: "2017-12-29T23:21:04+08:00"
draft: false
tags: ["izhouyan"]
categories: ["linux", "http"]
author: "izhouyan"

---

Http server's log is a treasure, you can use linux tools to find many interesting things.

Http server's log files normaly stay below dir /var/log/{nginx|apache}.

Cat the log file to read it.

> cat /var/log/nginx/access.log

File contens are like this.

> x.x.x.x - - [DD/MM/YYYY:HH:mm:ss -TZ] "GET /robots.txt HTTP/1.1" 200 57 "-" "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)"

It shows ip address, time, http method, url, status code, response length, client and so on line by line. You can config your http server log config to control log format and elements.

Simple task, how to get only the ip address?

> cat /var/log/nginx/access.log |cut -d ' ' -f 1

Using cut command, you can delimit the line and get specific field, the command above will just print every line's ip address.

Next, how to know which ip is our biggest client, and which is the second? We can sort it, like this:

> cat /var/log/nginx/access.log |cut -d ' ' -f 1|sort -n| uniq -c| sort -nr

This time we sort it first, and then uniq them, and sort it again to get a count and ip list.

If we want to know where this ip come from, we can write a script locate ip addres, cut the list, and provide ip addresses as parameters to run the script.

> cat /var/log/nginx/access.log |cut -d ' ' -f 1|sort -n| uniq -c| sort -nr > xargs locate-ip

That's simple.

With linux tools combined, adding some scripts, you could find what time your site is most busy, how many bytes your site respond, which url is visited most and so on, enjoy it.
