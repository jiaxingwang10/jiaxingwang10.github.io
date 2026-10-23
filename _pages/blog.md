---
layout: default
permalink: /blog/
title: blog
nav: true
nav_order: 1
pagination:
  enabled: true
  collection: posts
  permalink: /page/:num/
  per_page: 5
  sort_field: date
  sort_reverse: true
  trail:
    before: 1 # The number of links before the current page
    after: 3 # The number of links after the current page
---

<div class="post">

{% assign blog_name_size = site.blog_name | size %}
{% assign blog_description_size = site.blog_description | size %}

{% if blog_name_size > 0 or blog_description_size > 0 %}

  <div class="header-bar">
    <h1>{{ site.blog_name }}</h1>
    <h2>{{ site.blog_description }}</h2>
  </div>
  {% endif %}

{% assign designing_series = site.posts | where: "series", "designing-what-models-learn" | sort: "series_order" %}
{% if designing_series.size > 0 %}
  <section class="blog-series" aria-labelledby="designing-series-title">
    <div class="blog-series__header">
      <p class="blog-series__eyebrow">A THREE-PART SERIES</p>
      <h2 id="designing-series-title">Designing What Models Learn</h2>
      <p>Essays on targets, training distributions, and data operators.</p>
    </div>

    <div class="blog-series__list">
      {% for post in designing_series %}
        <article class="blog-series__card">
          <a href="{{ post.url | relative_url }}" aria-label="Read Part {{ post.series_order }}: {{ post.title }}">
            <div class="blog-series__body">
              <p class="blog-series__part">Part {{ post.series_order }} of {{ designing_series.size }}</p>
              <h3>{{ post.title }}</h3>
              <p>{{ post.description }}</p>
              {% assign read_time = post.content | strip_html | number_of_words | divided_by: 180 | plus: 1 %}
              <p class="blog-series__meta">
                {{ read_time }} min read &nbsp; &middot; &nbsp;
                <time datetime="{{ post.date | date: '%Y-%m-%d' }}">{{ post.date | date: '%B %d, %Y' }}</time>
              </p>
              <span class="blog-series__link">Read Part {{ post.series_order }} <span aria-hidden="true">→</span></span>
            </div>
          </a>
        </article>
      {% endfor %}
    </div>
  </section>

  <style>
    .blog-series {
      margin: 2.25rem 0 2.75rem;
      padding: clamp(1.25rem, 3vw, 2rem);
      border: 1px solid var(--global-divider-color);
      border-radius: 1rem;
      background: var(--global-bg-color);
    }

    .blog-series__header {
      max-width: 46rem;
      margin-bottom: 1.5rem;
    }

    .blog-series__eyebrow,
    .blog-series__part {
      margin: 0 0 0.35rem;
      color: var(--global-theme-color);
      font-size: 0.74rem;
      font-weight: 700;
      letter-spacing: 0.11em;
      text-transform: uppercase;
    }

    .blog-series__header h2 {
      margin: 0 0 0.4rem;
      color: var(--global-text-color);
      font-size: clamp(1.8rem, 4vw, 2.65rem);
      line-height: 1.1;
    }

    .blog-series__header > p:last-child {
      margin: 0;
      color: var(--global-text-color-light);
      font-size: 1.05rem;
    }

    .blog-series__list {
      display: grid;
      grid-template-columns: minmax(0, 1fr);
      gap: 1rem;
    }

    .blog-series__card {
      overflow: hidden;
      border: 1px solid var(--global-divider-color);
      border-radius: 0.8rem;
      background: var(--global-card-bg-color, var(--global-bg-color));
      transition: transform 160ms ease, border-color 160ms ease, box-shadow 160ms ease;
    }

    .blog-series__card:hover {
      transform: translateY(-3px);
      border-color: var(--global-theme-color);
      box-shadow: 0 0.75rem 1.75rem rgba(0, 0, 0, 0.10);
    }

    .blog-series__card > a {
      display: flex;
      height: 100%;
      flex-direction: column;
      color: inherit;
      text-decoration: none;
    }

    .blog-series__body {
      display: flex;
      flex: 1;
      flex-direction: column;
      padding: 1rem;
    }

    .blog-series__body h3 {
      margin: 0 0 0.65rem;
      color: var(--global-text-color);
      font-size: 1.12rem;
      line-height: 1.3;
      text-transform: none;
    }

    .blog-series__body > p:not(.blog-series__part):not(.blog-series__meta) {
      margin-bottom: 1rem;
      color: var(--global-text-color-light);
      font-size: 0.92rem;
      line-height: 1.5;
    }

    .blog-series__meta {
      margin: 0 0 0.75rem;
      color: var(--global-text-color-light);
      font-size: 0.8rem;
      line-height: 1.5;
    }

    .blog-series__link {
      margin-top: auto;
      color: var(--global-theme-color);
      font-size: 0.9rem;
      font-weight: 600;
    }

  </style>
{% endif %}

{% if site.display_tags and site.display_tags.size > 0 or site.display_categories and site.display_categories.size > 0 %}

  <div class="tag-category-list">
    <ul class="p-0 m-0">
      {% for tag in site.display_tags %}
        <li>
          <i class="fa-solid fa-hashtag fa-sm"></i> <a href="{{ tag | slugify | prepend: '/blog/tag/' | relative_url }}">{{ tag }}</a>
        </li>
        {% unless forloop.last %}
          <p>&bull;</p>
        {% endunless %}
      {% endfor %}
      {% if site.display_categories.size > 0 and site.display_tags.size > 0 %}
        <p>&bull;</p>
      {% endif %}
      {% for category in site.display_categories %}
        <li>
          <i class="fa-solid fa-tag fa-sm"></i> <a href="{{ category | slugify | prepend: '/blog/category/' | relative_url }}">{{ category }}</a>
        </li>
        {% unless forloop.last %}
          <p>&bull;</p>
        {% endunless %}
      {% endfor %}
    </ul>
  </div>
  {% endif %}

{% assign featured_posts = site.posts | where: "featured", "true" %}
{% if featured_posts.size > 0 %}
<br>

<div class="container featured-posts">
{% assign is_even = featured_posts.size | modulo: 2 %}
<div class="row row-cols-{% if featured_posts.size <= 2 or is_even == 0 %}2{% else %}3{% endif %}">
{% for post in featured_posts %}
<div class="col mb-4">
<a href="{{ post.url | relative_url }}">
<div class="card hoverable">
<div class="row g-0">
<div class="col-md-12">
<div class="card-body">
<div class="float-right">
<i class="fa-solid fa-thumbtack fa-xs"></i>
</div>
<h3 class="card-title text-lowercase">{{ post.title }}</h3>
<p class="card-text">{{ post.description }}</p>

                    {% if post.external_source == blank %}
                      {% assign read_time = post.content | number_of_words | divided_by: 180 | plus: 1 %}
                    {% else %}
                      {% assign read_time = post.feed_content | strip_html | number_of_words | divided_by: 180 | plus: 1 %}
                    {% endif %}
                    {% assign year = post.date | date: "%Y" %}

                    <p class="post-meta">
                      {{ read_time }} min read &nbsp; &middot; &nbsp;
                      <a href="{{ year | prepend: '/blog/' | relative_url }}">
                        <i class="fa-solid fa-calendar fa-sm"></i> {{ year }} </a>
                    </p>
                  </div>
                </div>
              </div>
            </div>
          </a>
        </div>
      {% endfor %}
      </div>
    </div>
    <hr>

{% endif %}

  <ul class="post-list">

    {% if page.pagination.enabled %}
      {% assign postlist = paginator.posts %}
    {% else %}
      {% assign postlist = site.posts %}
    {% endif %}

    {% for post in postlist %}

    {% if post.series == "designing-what-models-learn" %}
      {% continue %}
    {% endif %}

    {% if post.external_source == blank %}
      {% assign read_time = post.content | number_of_words | divided_by: 180 | plus: 1 %}
    {% else %}
      {% assign read_time = post.feed_content | strip_html | number_of_words | divided_by: 180 | plus: 1 %}
    {% endif %}
    {% assign year = post.date | date: "%Y" %}
    {% assign tags = post.tags | join: "" %}
    {% assign categories = post.categories | join: "" %}

    <li>

{% if post.thumbnail %}

<div class="row">
          <div class="col-sm-9">
{% endif %}
        <h3>
        {% if post.redirect == blank %}
          <a class="post-title" href="{{ post.url | relative_url }}">{{ post.title }}</a>
        {% elsif post.redirect contains '://' %}
          <a class="post-title" href="{{ post.redirect }}" target="_blank">{{ post.title }}</a>
          <svg width="2rem" height="2rem" viewBox="0 0 40 40" xmlns="http://www.w3.org/2000/svg">
            <path d="M17 13.5v6H5v-12h6m3-3h6v6m0-6-9 9" class="icon_svg-stroke" stroke="#999" stroke-width="1.5" fill="none" fill-rule="evenodd" stroke-linecap="round" stroke-linejoin="round"></path>
          </svg>
        {% else %}
          <a class="post-title" href="{{ post.redirect | relative_url }}">{{ post.title }}</a>
        {% endif %}
      </h3>
      <p>{{ post.description }}</p>
      <p class="post-meta">
        {{ read_time }} min read &nbsp; &middot; &nbsp;
        {{ post.date | date: '%B %d, %Y' }}
        {% if post.external_source %}
        &nbsp; &middot; &nbsp; {{ post.external_source }}
        {% endif %}
      </p>
      <p class="post-tags">
        <a href="{{ year | prepend: '/blog/' | relative_url }}">
          <i class="fa-solid fa-calendar fa-sm"></i> {{ year }} </a>

          {% if tags != "" %}
          &nbsp; &middot; &nbsp;
            {% for tag in post.tags %}
            <a href="{{ tag | slugify | prepend: '/blog/tag/' | relative_url }}">
              <i class="fa-solid fa-hashtag fa-sm"></i> {{ tag }}</a>
              {% unless forloop.last %}
                &nbsp;
              {% endunless %}
              {% endfor %}
          {% endif %}

          {% if categories != "" %}
          &nbsp; &middot; &nbsp;
            {% for category in post.categories %}
            <a href="{{ category | slugify | prepend: '/blog/category/' | relative_url }}">
              <i class="fa-solid fa-tag fa-sm"></i> {{ category }}</a>
              {% unless forloop.last %}
                &nbsp;
              {% endunless %}
              {% endfor %}
          {% endif %}
    </p>

{% if post.thumbnail %}

</div>

  <div class="col-sm-3">
    <img class="card-img" src="{{ post.thumbnail | relative_url }}" style="object-fit: cover; height: 90%" alt="image">
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
