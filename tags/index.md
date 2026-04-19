---
layout: default
title: Tags
permalink: /tags/
---

# All Tags

Explore content by tag. Click any tag below to see all posts and projects tagged with it.

<div class="tag-cloud">
{% comment %}
  Collect all unique tags from posts and projects
{% endcomment %}
{% assign all_tags = "" | split: "," %}

{% for post in site.posts %}
  {% for tag in post.tags %}
    {% assign all_tags = all_tags | push: tag %}
  {% endfor %}
{% endfor %}

{% for project in site.projects %}
  {% for tag in project.tags %}
    {% assign all_tags = all_tags | push: tag %}
  {% endfor %}
{% endfor %}

{% comment %}
  Remove duplicates and sort tags
{% endcomment %}
{% assign all_tags = all_tags | uniq | sort %}

{% comment %}
  For each unique tag, display it with counts
{% endcomment %}
<div class="tag-list">
{% for tag in all_tags %}
  {% assign post_count = 0 %}
  {% assign project_count = 0 %}
  
  {% for post in site.posts %}
    {% if post.tags contains tag %}
      {% assign post_count = post_count | plus: 1 %}
    {% endif %}
  {% endfor %}
  
  {% for project in site.projects %}
    {% if project.tags contains tag %}
      {% assign project_count = project_count | plus: 1 %}
    {% endif %}
  {% endfor %}
  
  {% assign total_count = post_count | plus: project_count %}
  
  <a href="/tags/{{ tag | slugify }}/" class="tag-item" data-count="{{ total_count }}">
    <span class="tag-name">{{ tag }}</span>
    <span class="tag-count">{{ total_count }}</span>
  </a>
{% endfor %}
</div>
