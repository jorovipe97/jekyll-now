---
layout: post
title: Hello LAMP
published: false
---

In this post are a lot of util snippets, tips and concepts for start with the LAMP stack acronym for (Linux, Apache, MySql, PHP)

This tutorial is done in Debian 8, but it works similarly in various unix systems.

# Installing the LAMP stack on Debian

Remember to update system's repository list before:
```bash
$ sudo apt update
```
And later install the updates:
```bash
$ sudo apt-get upgrade
```

## Installing Apache2
```bash
$ sudo apt-get install apache2
```


Starts the apache server:
```bash
$ sudo /etc/init.d/apache2 start
```
Stops the apache server:
```bash
$ sudo /etc/init.d/apache2 stop
```

Restarts the apache server:
```bash
$ sudo /etc/init.d/apache2 restart
```

## Installing MySql
```bash
$ sudo apt-get install mysql-server libapache2-mod-auth-mysql php5-mysql
```

## Installing PHP
```bash
$ sudo apt-get install php5
```

