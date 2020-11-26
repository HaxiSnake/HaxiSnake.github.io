---
title: django文档学习笔记
date: 2019-10-09 14:35:11
tags:
---

### Django改变模型的三步走


* 编辑 models.py 文件，改变模型。
* 运行 python manage.py makemigrations 为模型的改变生成迁移文件。
* 运行 python manage.py migrate 来应用数据库迁移。
