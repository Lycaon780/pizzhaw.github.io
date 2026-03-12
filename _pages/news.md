---
layout: default
title: news
meta_title: News | Pizzhaw CTF Team
permalink: /news/
nav: true
---

<div class="post">
  {% assign news_header_title = site.news_name | default: page.header_title %}
  {% assign news_header_description = site.news_description | default: page.header_description %}

  <div class="header-bar">
    <h1>{{ news_header_title }}</h1>
    <h2>{{ news_header_description }}</h2>
  </div>

  <ul class="post-list">
    {% if page.pagination.enabled %}
      {% assign newslist = paginator.posts %}
    {% else %}
      {% assign newslist = site.news | reverse %}
    {% endif %}

    {% for item in newslist %}
      <li>
        {% if item.thumbnail %}
          <div class="row">
            <div class="col-sm-9">
        {% endif %}

        <h3>
          <a class="post-title" href="{{ item.url | relative_url }}">{{ item.title }}</a>
        </h3>
        {% if item.description %}
          <p>{{ item.description }}</p>
        {% endif %}
        <p class="post-meta">{{ item.date | date: '%B %d, %Y' }}</p>

        {% if item.thumbnail %}
            </div>
            <div class="col-sm-3">
              <a href="{{ item.url | relative_url }}">
                <img class="card-img" src="{{ item.thumbnail | relative_url }}" style="object-fit: cover; height: 100%" alt="{{ item.title }} thumbnail" loading="lazy">
              </a>
            </div>
          </div>
        {% endif %}
      </li>
    {% endfor %}

  </ul>

{% if page.pagination.enabled %}
{% include pagination.liquid %}
{% endif %}

</div>
