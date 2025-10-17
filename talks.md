---
layout: page
title: Conferences and Talks
permalink: /talks/
---

{% assign items = site.talks | sort: "date" | reverse %}
<div class="post-list">
  {% for item in items %}
    {% comment %}
      Bild-URL aus Frontmatter holen:
      - unterstützt 'image: /pfad.jpg' (String)
      - und 'image: { path: /pfad.jpg }' (Objekt)
    {% endcomment %}
    {% assign img = item.image.path | default: item.image %}

    <article style="display:flex; align-items:flex-start; margin-bottom:30px; gap:20px;">
  {% if img %}
    <a href="{{ item.url | relative_url }}">
      <img src="{{ img | relative_url }}" alt="{{ item.title }}"
           style="width:160px; height:auto; border-radius:10px; box-shadow:0 2px 8px rgba(0,0,0,0.1);">
    </a>
  {% endif %}
  <div>
    <h2 class="post-title">
      <a href="{{ item.url | relative_url }}">{{ item.title }}</a>
    </h2>
    {% if item.date %}
      <p class="post-meta"><time datetime="{{ item.date | date_to_xmlschema }}">{{ item.date | date: "%Y-%m-%d" }}</time></p>
    {% endif %}
    {% if item.excerpt %}
      <p>{{ item.excerpt | strip_html | truncate: 160 }}</p>
    {% endif %}
  </div>
</article>
<hr>
  {% endfor %}
</div>