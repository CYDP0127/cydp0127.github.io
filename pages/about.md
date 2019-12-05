---
layout: page
title: About
description: Technologies are for changing the world
keywords: Daniel Choi, 최용득
comments: true
menu: About
permalink: /about/
---

> I believe life is a journey of finding meanings of life.

Hi, I am Daniel from South Korea, currently working at a startup company.
  
Life is a long journey of finding meanings of life.
Some are looking for a stable job and happy family and
some people value achieving career goals the most.

Regardless of what you are seeking, where you are, I hope you find the meanings of your life and most importantly, have a small break for a coffee sometimes :) ☕️

## Contact

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
