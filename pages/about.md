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

<img src="/images/About_Me.jpg" width="320" height="320" />

[Curriculum Vitae](./documents/Yuanzhi-Zhu-CV.pdf)

Research interests: Signal processing, Data science, Human-computer interaction, Spintronics

## Education

B.S. in Electronic and Information Engineering, Beihang University (Jul 2019)

B.S. Exchange in Electrical and Computer Engineering, Technical University of Munich(Oct 2019 - Mar 2020)

## Contact

{% for website in site.data.social %}
* {{ website.sitename }}: [@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skills

{% for category in site.data.skills %}
{{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}

## Useful Links

{% for link in site.data.links %}
  {% if link.src == 'www' %}
* [{{ link.name }}]({{ link.url }})
  {% endif %}
{% endfor %}

## Publication

{% for artical in site.data.publications %}
* {{ publications.authors }}{{ publications.name }}[{{ publications.available }}]({{ publications.url }})
{% endfor %}
