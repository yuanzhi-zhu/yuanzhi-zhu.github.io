---
layout: page
title: Recommended Sites
description: 
keywords: Recommended Sites
menu: ReadingList
permalink: /readinglist/
---

## Research Blogs (ML, AI, etc.)
<ul>
    {% for link in site.data.blogs %}
        {% if link.type == 'research' %}
            <li><a href="{{ link.url }}">{{ link.name }}</a> ({{ link.url }})</li>
        {% endif %}
    {% endfor %}
</ul>
