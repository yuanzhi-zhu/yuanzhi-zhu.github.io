---
layout: page
title: Publications
description: Full Publications
keywords: Publications
menu: Publications
permalink: /publications/
---

## Full Publications

#### Pre-prints
<ul>
    {% for link in site.data.full_pub %}
        {% if link.type == 'pre-print' %}
            <li><a href="{{ link.url }}">{{ link.name }}</a></li>
        {% endif %}
    {% endfor %}
</ul>

#### Conferences
<ul>
    {% for link in site.data.full_pub %}
        {% if link.type == 'conference' %}
            <li><a href="{{ link.url }}">{{ link.name }}</a></li>
        {% endif %}
    {% endfor %}
</ul>

#### Journals
<ul>
    {% for link in site.data.full_pub %}
        {% if link.type == 'journals' %}
            <li><a href="{{ link.url }}">{{ link.name }}</a></li>
        {% endif %}
    {% endfor %}
</ul>

