---
title: OSCP
layout: archive
permalink: /oscp/
classes: wide
---

Este listado incluye máquinas de HTB que pueden ser de gran ayuda si vas a hacer el OSCP.

<ul>
  {% for item in site.oscp %}
  <ul>
    <a href="{{ item.url | prepend: item.baseurl }}">{{ item.title }}</a>
  </ul>