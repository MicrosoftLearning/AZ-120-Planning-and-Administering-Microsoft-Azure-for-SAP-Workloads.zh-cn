---
title: 联机托管说明
permalink: index.html
layout: home
---

# 内容目录：AZ-120：计划和管理 Microsoft Azure for SAP 工作负载

所需的实验室文件可在[此处下载](https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads /archive/master.zip)

下面列出了每个实验室练习和演示的超链接。

## 实验室

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/'" %}
| 模块 | 实验室 |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
