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

<div style="display: flex; align-items: center;">
    <img src="/images/About_Me.jpg" width="320" height="320" alt="About Me" />
    <div style="margin-left: 50px; margin-right: 50px; padding: 20px 20px 20px 20px;">
        <p>I’m currently a Master's student at ETH Zürich, where I have the privilege of being supervised by <a href="https://cszn.github.io/">Dr. Kai Zhang</a> and <a href="https://apchenstu.github.io/">Dr. Anpei Chen</a>.</p>
        <p>My interests lie in Deep Learning and Computer Vision. Currently, my main areas of focus are Image Restoration and Novel View Synthesis.</p>
        <div style="display: flex; align-items: center;">
            {% for website in site.data.social %}
                <a href="{{ website.url }}" style="margin-right: 12px; background-color: white; display: inline-block;">
                    <img src="/assets/logos/{{ website.sitename }}.png" alt="{{ website.sitename }} logo" width="30" height="30" style="vertical-align: middle;"/>
                </a>
            {% endfor %}
        </div>
        <p><a href="/documents/Yuanzhi-Zhu-CV.pdf">Curriculum Vitae</a></p>
    </div>
</div>


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
