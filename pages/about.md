---
layout: page
title: About
description: It is never too LAZY to learn
keywords: Yuanzhi Zhu
menu: About
permalink: /about/
---

<style>
    @media (max-width: 600px) {
        .bio-container {
            flex-direction: column;
        }
        .bio-text {
            margin-left: 10px !important;
            margin-right: 10px !important;
        }
    }
    @media (max-width: 700px) {
        .my_container {
            flex-direction: column;
        }
    }
</style>

## About
<div class="bio-container" style="display: flex; align-items: center;">
    <div class="image-container" style="width: 256px; flex-shrink: 0;">
        <img src="/images/About_Me.jpg" width="256px" height=auto alt="About Me" />
        <p style="color:gray; font-size:0.7em">The person *NOT* wearing a blue cravat is me</p>
    </div>
    <div class="bio-text" style="margin-left: 50px; margin-right: 50px; padding: 20px 20px 20px 20px; text-align: justify;">
        <p style='margin-bottom: 5px;'>I’m Yuanzhi Zhu and I’m a PhD student at École Polytechnique, supervised by <a href="https://vicky.kalogeiton.info/">Prof. Vicky Kalogeiton</a> and <a href="https://stelat.eu/">Prof. Stéphane Lathuilière</a>. Previously I was a Master's student at ETH Zürich, where I have the privilege of being supervised by <a href="https://cszn.github.io/">Prof. Kai Zhang</a>. I also have the privilege of working as a RA with <a href="https://www.cs.utexas.edu/~lqiang/">Prof. Qiang Liu</a> and interned with <a href="https://hanshuyan.github.io/">Hanshu Yan</a> during my master's.</p>
        <!-- <p style='margin-bottom: 5px;'>My interests lie in Deep Learning and Computer Vision. Currently, my main areas of focus are Efficient Generative Models and Visual Enhancement.</p> -->
        <div id="extra-bio" style="display: none; margin-bottom: 5px;">
            <p style='margin-bottom: 5px;'>Previously I was a bachelor student in Beihang University, where I had the fortune of being mentored by both Dr. Zhizhong Zhang and Prof. Yue Zhang. Additionally, I had the opportunity to embark on a summer internship in <a href="https://otaniqnm.com/home/">Prof. Otani's lab</a>. I am from Quanjiao Middle School.</p>
        </div>
        <button id="show-more-button" style='margin-bottom: 10px;'>Show more</button>
        <div style="display: flex; align-items: center; margin-bottom: 10px;">
            {% for website in site.data.social %}
                <a href="{{ website.url }}" style="margin-right: 13.5px; background-color: white; display: inline-block;">
                    <img src="/assets/logos/{{ website.sitename }}.png" alt="{{ website.sitename }} logo" width="30" height="30" style="vertical-align: middle;"/>
                </a>
            {% endfor %}
        </div>
        <p style='margin-bottom: 5px;'><a href="/documents/Yuanzhi-Zhu-CV.pdf">CV</a></p>
        <p>firstname.lastname@polytechnique.edu</p>
    </div>
</div>

<div style="background:#f0f4ff; border-left:4px solid #3b82f6; padding:1rem; border-radius:0.5rem; margin:1rem 0;">
  Scalable generative models (AR, diffusion, etc.) aim to capture data distributions by decomposing likelihoods, inherently requiring multi-step inference. 
  The goal of efficient generative modeling is to uncover principles that enable high-quality generation with minimal steps.
</div>

<p style="color:rgb(254, 150, 15);">Do you see the future where human evolution enters cyber era -- or simply approaches extinction?</p>

## Selected Publications
{% for artical in site.data.conferences %}
* {{ artical.authors | replace: '[MY_NAME]', '<ins>Yuanzhi Zhu</ins>' | replace: '[EQUAL]', '\*' }} <br>
  {% if artical.website != null %} [**{{ artical.name }}**]({{ artical.website }}) {% else %} **{{ artical.name }}** {% endif %}
  {{ artical.publication }} {% if artical.arxiv_num %} *arxiv*: {{ artical.arxiv_num }} {% endif %}
  {% if artical.paper_url %} [ [paper] ]({{artical.paper_url }}) {% endif %} {% if artical.arxiv_url %} [ [arxiv] ]({{ artical.arxiv_url }}) {% endif %} {% if artical.code_url %} [ [code] ]({{ artical.code_url }}) {% endif %} {% if artical.slides_url %} [ [slides] ]({{ artical.slides_url }}) {% endif %} {% if artical.poster_url %} [ [poster] ]({{ artical.poster_url }}) {% endif %}
{% endfor %}

{% for artical in site.data.journals %}
* {{ artical.authors | replace: '[MY_NAME]', '<ins>Yuanzhi Zhu</ins>' | replace: '[EQUAL]', '\*' }} <br>
  **{{ artical.name }}**, {{artical.info}}[ [DOI: {{ artical.available }}] ]({{ artical.url }})
{% endfor %}

<div class="my_container" style="display: flex; justify-content: space-between;">
  <div class="my_column" style="flex: 1; margin-right: 75px;">
    <h2><a href="https://yuanzhi-zhu.github.io/readinglist/" style="text-decoration: none; color: inherit;">Recommended Sites</a></h2>
    <ul>
      <!-- only display the first three links -->
      {% for link in site.data.blogs limit:3 %}
      <!-- if link.type contains 'research' -->
        {% if link.type contains 'research' %}
          <li><a href="{{ link.url }}">{{ link.name }}</a></li>
        {% endif %}
      {% endfor %}
    </ul>
  </div>
  <div class="my_column" style="flex: 1;">
    <h2><a href="https://yuanzhi-zhu.github.io/documents/slides/" style="text-decoration: none; color: inherit;">Slides</a></h2>
    <ul>
      {% for link in site.data.slides %}
        {% if link.src == 'www' %}
          <li><a href="{{ link.url }}">{{ link.name }}</a></li>
        {% endif %}
      {% endfor %}
    </ul>
  </div>
</div>


## Honors and Awards
- CVPR 2025 outstanding reviewer

<!-- <div class="my_container" style="display: flex; justify-content: space-between;">
  <div class="my_column" style="flex: 1; margin-right: 75px;">
    <h2>Useful Links</h2>
    <ul>
      {% for link in site.data.links limit:3 %}
        {% if link.src == 'www' %}
          <li><a href="{{ link.url }}">{{ link.name }}</a></li>
        {% endif %}
      {% endfor %}
    </ul>
  </div>
  <div class="my_column" style="flex: 1;">
    <h2>Collaborators</h2>
    <ul>
      {% for link in site.data.collaborators %}
        {% if link.src == 'www' %}
          <li><a href="{{ link.url }}">{{ link.name }}</a></li>
        {% endif %}
      {% endfor %}
    </ul>
  </div>
</div> -->


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
