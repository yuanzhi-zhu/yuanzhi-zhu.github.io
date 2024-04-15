---
layout: page
title: Recommended Sites
description: 
keywords: Recommended Sites
menu: ReadingList
permalink: /readinglist/
---


{% for link in site.data.blogs %}
{% if link.src == 'www' %}
    <li><a href="{{ link.url }}">{{ link.name }}</a></li>
{% endif %}
{% endfor %}
