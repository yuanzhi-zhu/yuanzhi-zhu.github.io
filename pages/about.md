---
layout: page
title: About
description: It is never too LAZY to learn
keywords: Yuanzhi Zhu
comments: true
menu: About
permalink: /about/
---



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


<script type="text/javascript" id="clstr_globe" src="//cdn.clustrmaps.com/globe.js?d=5XlKTd1OHQI_bIzYLIz1ip29OgUjxc2bgXSB1W0mI28"></script>
