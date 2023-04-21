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

Research Interests: Image Restoration, Generative Models~~, Spintronics~~

## Education

B.S. in Electronic and Information Engineering, Beihang University (Sep 2016 - Jun 2020)

B.S. exchange in Electrical and Computer Engineering, Technical University of Munich (Oct 2019 - Mar 2020)

M.S in Information Technology and Electrical Engineering, Swiss Federal Institute of Technology in Zürich (ETHz) (Sep 2020 -)

## Contact

{% for website in site.data.social %}
* {{ website.sitename }}: [@{{ website.name }}]({{ website.url }})
{% endfor %}

## Publication
{% for artical in site.data.conferences %}
* {{ artical.authors | replace: '[MY_NAME]', '<ins>Yuanzhi Zhu</ins>' | replace: '[EQUAL]', '\*' }} <br>
  **{{ artical.name }}**
  {{ artical.publication }} {% if artical.arxiv_num %} *arxiv*: {{ artical.arxiv_num }} {% endif %}
  {% if artical.paper_url %} [ [paper] ]({{artical.paper_url }}) {% endif %} {% if artical.arxiv_url %} [ [arxiv] ]({{ artical.arxiv_url }}) {% endif %} {% if artical.code_url %} [ [code] ]({{ artical.code_url }}) {% endif %} {% if artical.slides_url %} [ [slides] ]({{ artical.slides_url }}) {% endif %}
{% endfor %}

{% for artical in site.data.journals %}
* {{ artical.authors | replace: '[MY_NAME]', '<ins>Yuanzhi Zhu</ins>' | replace: '[EQUAL]', '\*' }} <br>
  **{{ artical.name }}**, {{artical.info}}[ DOI: {{ artical.available }}]({{ artical.url }})
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
