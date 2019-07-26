---
layout: post
title: 'Hello GitBucket'
subtitle: '安装gitbucket'
date: 2019-06-15
author: Archer
categories: 版本控制
cover: ''
tags: gitbucket centos
---

```
Installation
GitBucket requires Java8. You have to install it, if it is not already installed.

Download the latest gitbucket.war from the releases page and run it by java -jar gitbucket.war.
Go to http://[hostname]:8080/ and log in with ID: root / Pass: root.
You can specify following options:

--port=[NUMBER]
--prefix=[CONTEXTPATH]
--host=[HOSTNAME]
--gitbucket.home=[DATA_DIR]
--temp_dir=[TEMP_DIR]
--max_file_size=[MAX_FILE_SIZE]
TEMP_DIR is used as the temporary directory for the jetty application context. This is the directory into which the gitbucket.war file is unpacked, the source files are compiled, etc. If given this parameter must match the path of an existing directory or the application will quit reporting an error; if not given the path used will be a tmp directory inside the gitbucket home.

MAX_FILE_SIZE is the max file size for upload files.

You can also deploy gitbucket.war to a servlet container which supports Servlet 3.0 (like Jetty, Tomcat, JBoss, etc)

For more information about installation on Mac or Windows Server (with IIS), or configuration of Apache or Nginx and also integration with other tools or services such as Jenkins or Slack, see Wiki.

To upgrade GitBucket, replace gitbucket.war with the new version, after stopping GitBucket. All GitBucket data is stored in HOME/.gitbucket by default. So if you want to back up GitBucket's data, copy this directory to the backup location.
```
