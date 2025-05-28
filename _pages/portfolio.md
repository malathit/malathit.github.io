---
title: "Portfolio"
permalink: /portfolio/
layout: single
author_profile: true
---
{% if jekyll.environment == "production" %}
  {% assign flags = site.data.feature_flags.production %}
{% else %}
  {% assign flags = site.data.feature_flags.development %}
{% endif %}

{% if flags.portfolio.enabled %}
## My Projects

Here are some of the projects I've worked on:

{% for project in site.portfolio %}
  <div class="project-card">
    <h2>{{ project.title }}</h2>
    {% if project.image %}
      <img src="{{ project.image }}" alt="{{ project.title }}" class="project-image">
    {% endif %}
    <p>{{ project.description }}</p>
    {% if project.technologies %}
      <div class="technologies">
        <strong>Technologies:</strong> {{ project.technologies | join: ", " }}
      </div>
    {% endif %}
    {% if project.github %}
      <a href="{{ project.github }}" class="btn btn--primary">View on GitHub</a>
    {% endif %}
    {% if project.demo %}
      <a href="{{ project.demo }}" class="btn btn--success">Live Demo</a>
    {% endif %}
  </div>
{% endfor %}
{% else %}
  <p>Portfolio is currently under maintenance. Please check back later!</p>
{% endif %}