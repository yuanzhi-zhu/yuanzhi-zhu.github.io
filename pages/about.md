---
layout: page
title: About
description: It is never too LAZY to learn
keywords: Yuanzhi Zhu
comments: true
menu: About
permalink: /about/
---

## About

[{{ Curriculum Vitae }}]({{ /documents/Yuanzhi-Zhu-CV.pdf }})

Research interests: Data science, Image processing, Human-computer interaction, Spintronics


## Education

B.S. in Electronic, Beihang University (Jul 2019)

## Contact

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skills

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}

## Global Map
<script type="text/javascript" id="clustrmaps" src="//cdn.clustrmaps.com/map_v2.js?d=5XlKTd1OHQI_bIzYLIz1ip29OgUjxc2bgXSB1W0mI28&cl=ffffff&w=a"></script>
