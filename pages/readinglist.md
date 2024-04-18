---
layout: page
title: Recommended Sites
description: 
keywords: Recommended Sites
menu: ReadingList
permalink: /readinglist/
---

## Research

#### Blogs
<ul>
    {% for link in site.data.blogs %}
        {% if link.type == 'research_blog' %}
            <li><a href="{{ link.url }}">{{ link.name }}</a> ({{ link.url }})</li>
        {% endif %}
    {% endfor %}
</ul>

#### Slides
<ul>
    {% for link in site.data.blogs %}
        {% if link.type == 'research_slides' %}
            <li><a href="{{ link.url }}">{{ link.name }}</a> ({{ link.url }})</li>
        {% endif %}
    {% endfor %}
</ul>

## Useful Links

<ul>
    {% for link in site.data.links %}
        {% if link.src == 'www' %}
            <li><a href="{{ link.url }}">{{ link.name }}</a></li>
        {% endif %}
    {% endfor %}
</ul>


