---
title: Instructions hébergées en ligne
permalink: index.html
layout: home
---

# Exercices sur l’administration de bases de données

Ces exercices prennent en charge le cours Microsoft [DP-300 : Administration des solutions Microsoft Azure SQL](https://docs.microsoft.com/training/courses/dp-300t00).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Module | Exercice |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

