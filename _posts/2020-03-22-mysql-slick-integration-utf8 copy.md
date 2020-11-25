---
layout: post
title: MySQL & Slick Integration w/ UTF-8
author: Fahad Siddiqui
authorUrl: https://github.com/fahadsiddiqui
date: "2020-03-22 00:21:49"
---

## Slick Part

To support UTF-8 through Slick, we need to put `characterEncoding=UTF-8` in JDBC string

Such as,

```
url = "jdbc:mysql://"${MYSQL_HOST}"/discoverer_db?autoReconnect=true&useSSL=false&characterEncoding=UTF-8"
```

For example, the final Slick configuration will be

```
slick {
  dbs {
    default {
      profile = "slick.jdbc.MySQLProfile$"
      driver = "com.mysql.cj.jdbc.Driver"
      db {
        driver = "com.mysql.cj.jdbc.Driver"
        url = "jdbc:mysql://"${MYSQL_HOST}"/discoverer_db?autoReconnect=true&useSSL=false&characterEncoding=UTF-8"
        user = "discoverer_user"
        password = "password1234567890"
      }
    }
  }
}
```

## Docker Compose Part

Docker image needs to know when running what character encoding is to be set, to do this we add `command:` in `.yml` as

```yaml
command:
  --default-authentication-plugin=mysql_native_password --character-set-server=utf8 --collation-server=utf8_unicode_ci
```

This will turn on the character encoding to UTF-8.

For example, the `docker-compose.yml` looks like

```yaml
version: '3.3'
services:
  gemi_api:
    ...
  discoverer_db:
    container_name: 'discoverer_db'
    image: mysql:5.7

    command:
      --default-authentication-plugin=mysql_native_password --character-set-server=utf8 --collation-server=utf8_unicode_ci

    ...
```
