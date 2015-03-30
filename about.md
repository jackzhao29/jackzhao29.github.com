---
title: About
layout: page
group: navigation
comment: true
---

#### 花名：关心
#### 关注：java、scala、java performance、分布式、高并发
#### Email: jack.zhao829@gmail.com

#### Contact me

{% if site.author.email %}
Email：{{ site.author.email }}
{% endif %}

{% if site.author.weibo %}
Weibo：<http://weibo.com/{{ site.author.weibo }}>
{% endif %}

{% if site.author.github %}
Github：<https://github.com/{{ site.author.github }}>
{% endif %}

{% if site.author.twitter %}
Twitter：<https://twitter.com/{{ site.author.twitter }}>
{% endif %}

{% if site.url %}
RSS：[{{ site.url }}{{ '/rss.xml' }}](/rss.xml)
{% endif %}

{% include support.html %}

{% include comments.html %}
