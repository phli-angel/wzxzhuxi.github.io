---
layout: page
title: 作者
permalink: /authors/
---

<section class="author-search">
  <label for="author-search-input">按作者或标题筛选文章</label>
  <input type="text" id="author-search-input" placeholder="输入作者姓名或文章标题，例如 老王 或 数据库索引" />
</section>

{% assign grouped_authors = site.posts | group_by: "author" | sort: "name" %}

<div class="authors-grid" id="authors-list">
  {% for author_group in grouped_authors %}
    {% assign author_key = author_group.name %}
    {% assign author_data = site.data.authors[author_key] %}
    <article class="author-card" data-author="{{ author_key | downcase }}">
      <div class="author-header">
        <p class="author-label">作者</p>
        <h2>
          {% if author_data %}
            {{ author_data.name }}
          {% else %}
            {{ author_key }}
          {% endif %}
        </h2>
        <p class="author-count">{{ author_group.items | size }} 篇文章</p>
      </div>
      <div class="posts-grid">
        {% for post in author_group.items %}
          <article class="post-card"
                   data-post-title="{{ post.title | downcase | escape }}"
                   data-post-author="{{ post.author | default: author.name | downcase | escape }}">
            <div class="post-number">{{ forloop.index | prepend: '00' | slice: -2, 2 }}</div>
            <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
            <p class="post-meta">{{ post.date | date: "%Y-%m-%d" }}</p>
            <p class="post-excerpt">{{ post.excerpt | strip_html | truncate: 160 }}</p>
            <a href="{{ post.url }}" class="read-more">阅读文章 →</a>
          </article>
        {% endfor %}
      </div>
    </article>
  {% endfor %}
</div>

<p class="author-empty" data-empty-state>找不到匹配的作者，请尝试其他关键词。</p>

<style>
  .author-search {
    margin-bottom: 2rem;
    border: 4px solid #000;
    padding: 2rem;
    background: #fff;
  }

  .author-search label {
    display: block;
    font-size: 0.85rem;
    letter-spacing: 0.2em;
    text-transform: uppercase;
    margin-bottom: 1rem;
    opacity: 0.7;
  }

  .author-search input {
    width: 100%;
    padding: 1rem 1.5rem;
    border: 3px solid #000;
    font-size: 1rem;
    font-family: inherit;
  }

  .authors-grid {
    display: flex;
    flex-direction: column;
    gap: 3rem;
  }

  .author-card {
    border: 4px solid #000;
    padding: 2.5rem;
    background: #fff;
    transition: transform 0.3s ease;
  }

  .author-card:hover {
    transform: translate(-6px, -6px);
    box-shadow: 12px 12px 0 #000;
  }

  .author-label {
    font-size: 0.85rem;
    letter-spacing: 0.2em;
    text-transform: uppercase;
    opacity: 0.6;
  }

  .author-card h2 {
    font-size: 2.2rem;
    margin: 0.5rem 0;
  }

  .author-count {
    font-size: 0.9rem;
    letter-spacing: 0.1em;
    opacity: 0.7;
    text-transform: uppercase;
  }

  .author-role {
    font-size: 1.2rem;
    font-weight: 400;
    opacity: 0.8;
  }

  .author-bio {
    margin-top: 1rem;
    font-size: 1rem;
    line-height: 1.6;
    opacity: 0.85;
  }

  .authors-grid .posts-grid {
    display: grid;
    gap: 2rem;
    margin-top: 2rem;
  }

  .authors-grid .post-card {
    border: 4px solid #000;
    padding: 2rem;
    position: relative;
    background: #fff;
  }

  .authors-grid .post-number {
    position: absolute;
    top: -1.5rem;
    right: 1.5rem;
    background: #000;
    color: #fff;
    padding: 0.4rem 1.2rem;
    font-weight: 900;
  }

  .authors-grid .post-card h3 {
    font-size: 1.6rem;
    margin-top: 0;
  }

  .authors-grid .post-card h3 a {
    color: #000;
    text-decoration: none;
    border-bottom: 4px solid transparent;
    transition: border-color 0.2s ease, color 0.2s ease, background-color 0.2s ease;
  }

  .authors-grid .post-card h3 a:hover {
    border-bottom-color: #000;
    background: transparent;
    color: #000;
    padding: 0;
  }

  .author-empty {
    display: none;
    margin-top: 2rem;
    font-weight: 700;
    letter-spacing: 0.1em;
    text-transform: uppercase;
  }

  @media (max-width: 768px) {
    .authors-grid .posts-grid {
      grid-template-columns: 1fr;
    }
  }
</style>

<script>
  (function() {
    const input = document.getElementById('author-search-input');
    const cards = Array.from(document.querySelectorAll('.author-card'));
    const emptyState = document.querySelector('[data-empty-state]');

    function showAllPosts(card, posts) {
      posts.forEach(post => {
        post.style.display = 'block';
      });
    }

    function filterAuthors() {
      const query = input.value.trim().toLowerCase();
      let anyVisible = false;

      cards.forEach(card => {
        const posts = Array.from(card.querySelectorAll('.post-card'));
        const authorName = (card.dataset.author || '').toLowerCase();

        if (!query) {
          card.style.display = 'block';
          showAllPosts(card, posts);
          anyVisible = true;
          return;
        }

        const authorMatch = authorName.includes(query);

        if (authorMatch) {
          card.style.display = 'block';
          showAllPosts(card, posts);
          anyVisible = true;
          return;
        }

        let matchedPosts = 0;

        posts.forEach(post => {
          const titleValue = (post.dataset.postTitle || '').toLowerCase();
          const postAuthorValue = (post.dataset.postAuthor || authorName).toLowerCase();
          const matches = titleValue.includes(query) || postAuthorValue.includes(query);
          post.style.display = matches ? 'block' : 'none';
          if (matches) matchedPosts++;
        });

        if (matchedPosts > 0) {
          card.style.display = 'block';
          anyVisible = true;
        } else {
          card.style.display = 'none';
        }
      });

      if (emptyState) {
        emptyState.style.display = anyVisible ? 'none' : 'block';
      }
    }

    input.addEventListener('input', filterAuthors);
    filterAuthors();
  })();
</script>
