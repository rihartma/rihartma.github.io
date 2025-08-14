---
layout: default
title: Home
---

I started this blog mostly for me. Whenever a cool idea pops into my head, writing it down pushes me to think it through more clearly and deeply.
You'll find my random thoughts here on stuff I find interesting—mostly theoretical computer science, quantum computing,math, and things like that.

Below are my latest posts:

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url }}) - {{ post.date | date: "%B %d, %Y" }}
{% endfor %}
