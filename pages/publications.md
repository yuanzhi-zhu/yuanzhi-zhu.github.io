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
    {% for artical in site.data.full_pub %}
        {% if artical.type == 'pre-print' %}
            * {{ artical.authors | replace: '[MY_NAME]', '<ins>Yuanzhi Zhu</ins>' | replace: '[EQUAL]', '\*' }} <br>
            {% if artical.website != null %} [**{{ artical.name }}**]({{ artical.website }}) {% else %} **{{ artical.name }}** {% endif %}
            {{ artical.publication }} {% if artical.arxiv_num %} *arxiv*: {{ artical.arxiv_num }} {% endif %}
            {% if artical.paper_url %} [ [paper] ]({{artical.paper_url }}) {% endif %} {% if artical.arxiv_url %} [ [arxiv] ]({{ artical.arxiv_url }}) {% endif %} {% if artical.code_url %} [ [code] ]({{ artical.code_url }}) {% endif %} {% if artical.slides_url %} [ [slides] ]({{ artical.slides_url }}) {% endif %} {% if artical.poster_url %} [ [poster] ]({{ artical.poster_url }}) {% endif %}
        {% endif %}
    {% endfor %}
</ul>

#### Conferences
<ul>
    {% for artical in site.data.full_pub %}
        {% if artical.type == 'conference' %}
            * {{ artical.authors | replace: '[MY_NAME]', '<ins>Yuanzhi Zhu</ins>' | replace: '[EQUAL]', '\*' }} <br>
            {% if artical.website != null %} [**{{ artical.name }}**]({{ artical.website }}) {% else %} **{{ artical.name }}** {% endif %}
            {{ artical.publication }} {% if artical.arxiv_num %} *arxiv*: {{ artical.arxiv_num }} {% endif %}
            {% if artical.paper_url %} [ [paper] ]({{artical.paper_url }}) {% endif %} {% if artical.arxiv_url %} [ [arxiv] ]({{ artical.arxiv_url }}) {% endif %} {% if artical.code_url %} [ [code] ]({{ artical.code_url }}) {% endif %} {% if artical.slides_url %} [ [slides] ]({{ artical.slides_url }}) {% endif %} {% if artical.poster_url %} [ [poster] ]({{ artical.poster_url }}) {% endif %}
        {% endif %}
    {% endfor %}
</ul>

#### Journals
<ul>
    {% for artical in site.data.full_pub %}
        {% if artical.type == 'journal' %}
            * {{ artical.authors | replace: '[MY_NAME]', '<ins>Yuanzhi Zhu</ins>' | replace: '[EQUAL]', '\*' }} <br>
            {% if artical.website != null %} [**{{ artical.name }}**]({{ artical.website }}) {% else %} **{{ artical.name }}** {% endif %}
            {{ artical.publication }} {% if artical.arxiv_num %} *arxiv*: {{ artical.arxiv_num }} {% endif %}
            {% if artical.paper_url %} [ [paper] ]({{artical.paper_url }}) {% endif %} {% if artical.arxiv_url %} [ [arxiv] ]({{ artical.arxiv_url }}) {% endif %} {% if artical.code_url %} [ [code] ]({{ artical.code_url }}) {% endif %} {% if artical.slides_url %} [ [slides] ]({{ artical.slides_url }}) {% endif %} {% if artical.poster_url %} [ [poster] ]({{ artical.poster_url }}) {% endif %}
        {% endif %}
    {% endfor %}
</ul>

