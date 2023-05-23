---
layout: page
title: About
description: It is never too LAZY to learn
keywords: Yuanzhi Zhu
menu: About
permalink: /about/
---

## About
<style>
    @media (max-width: 600px) {
        .bio-container {
            flex-direction: column;
        }
        .bio-text {
            margin-left: 0;
        }
    }
</style>
<div class="bio-container" style="display: flex; align-items: center;">
    <div class="image-container" style="width: 256; flex-shrink: 0;">
        <img src="/images/About_Me.jpg" width="256" height=auto alt="About Me" />
        <p style="color:gray; font-size:0.7em">The person *NOT* wearing a blue cravat is me</p>
    </div>
    <div class="bio-text" style="margin-left: 50px; margin-right: 50px; padding: 20px 20px 20px 20px;">
        <p style='margin-bottom: 5px;'>I’m Yuanzhi Zhu and I’m currently a Master's student at ETH Zürich, where I have the privilege of being supervised by <a href="https://cszn.github.io/">Dr. Kai Zhang</a> and <a href="https://apchenstu.github.io/">Dr. Anpei Chen</a>.</p>
        <p style='margin-bottom: 5px;'>My interests lie in Deep Learning and Computer Vision. Currently, my main areas of focus are Image Restoration and Novel View Synthesis.</p>
        <div id="extra-bio" style="display: none; margin-bottom: 5px;">
            <p style='margin-bottom: 5px;'>Previously I was a bachelor student in Beihang University, where I had the fortune of being mentored by both Dr. Zhizhong Zhang and Prof. Yue Zhang. Additionally, I had the opportunity to embark on a summer internship in <a href="https://otaniqnm.com/home/">Prof. Otani's lab</a>.</p>
        </div>
        <button id="show-more-button" style='margin-bottom: 10px;'>Show more</button>
        <div style="display: flex; align-items: center;">
            {% for website in site.data.social %}
                <a href="{{ website.url }}" style="margin-right: 13.5px; background-color: white; display: inline-block;">
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

<div class="container" style="margin-left: -30px;">
  <div class="column" style="margin-right: 150px;">
    <h2>Useful Links</h2>
    <ul>
      {% for link in site.data.links %}
        {% if link.src == 'www' %}
          <li><a href="{{ link.url }}">{{ link.name }}</a></li>
        {% endif %}
      {% endfor %}
    </ul>
  </div>
  <div class="column">
    <h2>Slides</h2>
    <ul>
      {% for link in site.data.slides %}
        {% if link.src == 'www' %}
          <li><a href="{{ link.url }}">{{ link.name }}</a></li>
        {% endif %}
      {% endfor %}
    </ul>
  </div>
</div>

<script>
document.getElementById("show-more-button").addEventListener("click", function() {
    var extraBio = document.getElementById("extra-bio");
    var button = document.getElementById("show-more-button");
    if (extraBio.style.display === "none") {
        extraBio.style.display = "block";
        button.textContent = "Show less"; // change button text when extra bio is displayed
    } else {
        extraBio.style.display = "none";
        button.textContent = "Show more"; // change button text when extra bio is hidden
    }
});
</script>
