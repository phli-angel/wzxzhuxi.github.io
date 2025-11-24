---
layout: page
title: 书籍
permalink: /books/
---

<section class="books-collection">
  <div class="posts-grid">
    {% assign all_books = "" | split: "" %}
    {% for item in site.books %}
      {% unless item.book or item.path contains 'TEMPLATE' %}
        {% assign all_books = all_books | push: item %}
      {% endunless %}
    {% endfor %}
    {% assign all_books = all_books | sort: "order" %}
    {% for book in all_books %}
      <article class="post-card">
        <div class="post-number">BK{{ forloop.index | prepend: '0' | slice: -2, 2 }}</div>
        <h2>
          <a href="{{ book.url }}">{{ book.title }}</a>
        </h2>
        {% assign book_author_data = site.data.authors[book.author] %}
        <p class="post-meta">
          多章节长文 · 作者：
          {% if book_author_data %}
            {{ book_author_data.name }}
          {% else %}
            {{ book.author | default: "A_Bit_S 团队" }}
          {% endif %}
        </p>
        <p class="post-excerpt">{{ book.description | strip_html | truncate: 220 }}</p>
        <a href="{{ book.url }}" class="read-more">查看章节 →</a>
      </article>
    {% endfor %}
  </div>
</section>

<style>
  .books-collection .posts-grid {
    display: grid;
    gap: 3rem;
  }

  .books-collection .post-card {
    border: 4px solid #000;
    padding: 3rem;
    position: relative;
    transition: all 0.3s;
    background: #fff;
  }

  .books-collection .post-card:hover {
    box-shadow: 12px 12px 0 #000;
    transform: translate(-6px, -6px);
  }
</style>
