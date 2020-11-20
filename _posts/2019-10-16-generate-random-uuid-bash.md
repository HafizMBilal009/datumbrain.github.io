---
layout: post
title: Generate Random UUID Using Shell
author: Fahad Siddiqui
authorUrl: https://github.com/fahadsiddiqui
date: "2019-10-16 23:42:00"
---

## /dev/urandom

Quoting die (dot) net:

> The character special files /dev/random and /dev/urandom (present since Linux 1.3.30) provide an interface to the kernel's random number generator. File /dev/random has major device number 1 and minor device number 8. File /dev/urandom has major device number 1 and minor device number 9.

We can use either `/dev/urandom` or `/dev/random` to achieve this purpose.

```bash
$ cat /dev/urandom
```

This gives us content of the file. Let's pipe this into `tr` (translate or delete characters, checkout the [man](http://linuxcommand.org/lc3_man_pages/tr1.html) page for more details; go deeper) to filter out only lower-case, upper-case letters and numbers. 

```bash
$ cat /dev/urandom | tr -dc 'a-zA-Z0-9'
```

Now we need only 32 characters out of these filtered characters. Let's use `fold`.

```bash
$ cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1
```

And at the end, just to make sure we find the first out of many filtered strings, we used `head` here.

So this will give you a UUID, always guaranteed random by your OS.

You can store in some variable using legacy bourne shell backticks ``` `` ``` or `$()` operator.



```bash
$ NEW_UUID=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1)
$ echo $NEW_UUID
S23nWnMLnmnnpTf6cCkQG5MqvUuqMLwo
```

Thanks for staying till the end. Here's a bashato for you.