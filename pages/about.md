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

[Curriculum Vitae](/documents/Yuanzhi-Zhu-CV.pdf)

Research Interests: Computer Vision, Natural Language Processing, Spintronics

## Education

B.S. in Electronic and Information Engineering, Beihang University (Jul 2019)

B.S. exchange in Electrical and Computer Engineering, Technical University of Munich(Oct 2019 - Mar 2020)

M.S in Information Technology and Electrical Engineering, Swiss Federal Institute of Technology in Zürich(ETH)(Sep 2020 -)

## Contact

{% for website in site.data.social %}
* {{ website.sitename }}: [@{{ website.name }}]({{ website.url }})
{% endfor %}

## Publication

{% for artical in site.data.publications %}
* {{ artical.authors }}
  {{ artical.name }}[ DOI: {{ artical.available }}]({{ artical.url }})
{% endfor %}

## Useful Links

{% for link in site.data.links %}
  {% if link.src == 'www' %}
* [{{ link.name }}]({{ link.url }})
  {% endif %}
{% endfor %}

## Slides

{% for link in site.data.slides %}
  {% if link.src == 'www' %}
* [{{ link.name }}]({{ link.url }})
  {% endif %}
{% endfor %}
