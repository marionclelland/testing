---
layout: default
title: "Critical Alerts Index"
permalink: /alerts/home/
---

{% for item in site.alerts %}
  {% if item.url contains '_prod' %}
    <h2>{{ item.title }}</h2>
    <p><a href="{{ item.url }}">{{ item.title }}</a></p>
  {% endif %}
{% endfor %}
