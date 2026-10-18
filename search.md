---
layout: page
title: 搜索
permalink: /search/
description: 纯前端搜索，数据来自 /search.json，不依赖任何第三方服务。
---

<p class="search-hint">输入关键词即可实时过滤标题、摘要、标签与正文。支持多个关键词（空格分隔）。</p>

<div class="search-box">
  <input type="search" id="search-input" class="search-input" placeholder="例如：MCP、评测、Spring Boot …" autocomplete="off" autofocus>
  <span class="search-status" id="search-status"></span>
</div>

<div id="search-results" class="search-results"></div>

<script>
(function () {
  var input = document.getElementById('search-input');
  var results = document.getElementById('search-results');
  var status = document.getElementById('search-status');
  var all = [];
  var loaded = false;

  function escapeHtml(s) {
    return String(s).replace(/[&<>"']/g, function (c) {
      return ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' })[c];
    });
  }

  function highlight(text, terms) {
    var safe = escapeHtml(text || '');
    terms.forEach(function (t) {
      if (!t) return;
      var re = new RegExp('(' + t.replace(/[.*+?^${}()|[\]\\]/g, '\\$&') + ')', 'gi');
      safe = safe.replace(re, '<mark>$1</mark>');
    });
    return safe;
  }

  function snippet(content, terms) {
    if (!content) return '';
    var lower = content.toLowerCase();
    var idx = -1;
    for (var i = 0; i < terms.length && idx < 0; i++) {
      idx = lower.indexOf(terms[i]);
    }
    if (idx < 0) return content.slice(0, 120) + '…';
    var start = Math.max(0, idx - 40);
    return (start > 0 ? '…' : '') + content.slice(start, start + 140) + '…';
  }

  function render(html) { results.innerHTML = html; }

  function search(q) {
    var terms = q.toLowerCase().split(/\s+/).filter(Boolean);
    if (!terms.length) {
      status.textContent = all.length ? '共 ' + all.length + ' 篇文章可搜索' : '';
      render(all.map(function (p) { return card(p, []); }).join(''));
      return;
    }
    var hits = all.filter(function (p) {
      var hay = (p.title + ' ' + p.excerpt + ' ' + p.content + ' ' + p.categories.join(' ') + ' ' + p.tags.join(' ')).toLowerCase();
      return terms.every(function (t) { return hay.indexOf(t) >= 0; });
    });
    status.textContent = '找到 ' + hits.length + ' 篇';
    if (!hits.length) {
      render('<p class="empty-hint">没有匹配的文章，换个关键词试试。</p>');
      return;
    }
    render(hits.map(function (p) { return card(p, terms); }).join(''));
  }

  function card(p, terms) {
    var tags = p.tags.map(function (t) { return '<span class="tag-chip">#' + escapeHtml(t) + '</span>'; }).join('');
    var cats = p.categories.map(function (c) { return '<span class="cat-chip">' + escapeHtml(c) + '</span>'; }).join('');
    return '<article class="search-item">' +
      '<p class="post-meta">' + escapeHtml(p.date) + '</p>' +
      '<h3 class="search-item-title"><a href="' + p.url + '">' + highlight(p.title, terms) + '</a></h3>' +
      '<p class="search-item-snippet">' + highlight(snippet(p.excerpt || p.content, terms), terms) + '</p>' +
      '<p class="search-item-meta">' + cats + tags + '</p>' +
    '</article>';
  }

  if (!input) return;
  status.textContent = '正在加载索引…';

  fetch('{{ "/search.json" | relative_url }}')
    .then(function (r) { return r.json(); })
    .then(function (data) {
      all = data;
      loaded = true;
      status.textContent = '共 ' + all.length + ' 篇文章可搜索';
      var q = new URLSearchParams(window.location.search).get('q');
      if (q) { input.value = q; }
      search(input.value || '');
    })
    .catch(function () {
      status.textContent = '索引加载失败，请刷新重试。';
    });

  var timer;
  input.addEventListener('input', function () {
    clearTimeout(timer);
    var v = input.value;
    timer = setTimeout(function () { if (loaded) search(v); }, 120);
  });

  input.addEventListener('keydown', function (e) {
    if (e.key === 'Enter') { if (loaded) search(input.value); }
  });
})();
</script>
