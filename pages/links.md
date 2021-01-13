---
layout: page
title: Links
description:  
keywords: Links
comments: true
menu: Link
permalink: /links/
---

> Friends

{% for link in site.data.links %}
  {% if link.src == 'life' %}

* [{{ link.name }}]({{ link.url }})
  {% endif %}
{% endfor %}

> Useful Links

{% for link in site.data.links %}
  {% if link.src == 'www' %}
* [{{ link.name }}]({{ link.url }})
  {% endif %}
{% endfor %}
