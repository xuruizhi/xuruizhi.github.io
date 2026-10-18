---
layout: page
title: 搜索
subtitle: 在全部文章中查找
permalink: /search/
---

<div class="search-page">
  <input type="search" id="search-input" class="search-input" placeholder="输入关键词，例如 MCP、评测、Git …" autocomplete="off" autofocus>
  <p class="search-status" id="search-status">正在加载索引…</p>
  <div id="search-results" class="search-results"></div>
</div>

<script>
(function () {
  var input = document.getElementById('search-input');
  var status = document.getElementById('search-status');
  var results = document.getElementById('search-results');
  var all = [];

  function esc(s) {
    return String(s == null ? '' : s).replace(/[&<>"']/g, function (c) {
      return ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' })[c];
    });
  }

  function highlight(text, terms) {
    var out = esc(text);
    terms.forEach(function (t) {
      if (!t) return;
      var re = new RegExp('(' + t.replace(/[.*+?^${}()|[\]\\]/g, '\\$&') + ')', 'gi');
      out = out.replace(re, '<mark>$1</mark>');
    });
    return out;
  }

  function snippet(content, terms) {
    if (!content) return '';
    var lower = content.toLowerCase();
    var idx = -1;
    for (var i = 0; i < terms.length && idx < 0; i++) idx = lower.indexOf(terms[i]);
    if (idx < 0) return content.slice(0, 150) + '…';
    var start = Math.max(0, idx - 50);
    return (start > 0 ? '…' : '') + content.slice(start, start + 170) + '…';
  }

  function card(p, terms) {
    var tags = (p.tags || []).map(function (t) { return '<span class="tag-chip">#' + esc(t) + '</span>'; }).join('');
    return '<article class="search-item">' +
      '<p class="post-meta">' + esc(p.date) + '</p>' +
      '<h2 class="post-title"><a href="' + p.url + '">' + highlight(p.title, terms) + '</a></h2>' +
      '<p class="search-snippet">' + highlight(snippet(p.excerpt || p.content, terms), terms) + '</p>' +
      '<p class="search-tags">' + tags + '</p>' +
      '</article>';
  }

  function run(q) {
    var terms = (q || '').toLowerCase().split(/\s+/).filter(Boolean);
    if (!terms.length) {
      status.textContent = '共 ' + all.length + ' 篇文章可搜索';
      results.innerHTML = all.map(function (p) { return card(p, []); }).join('');
      return;
    }
    var hits = all.filter(function (p) {
      var hay = (p.title + ' ' + (p.excerpt || '') + ' ' + (p.content || '') + ' ' + (p.tags || []).join(' ')).toLowerCase();
      return terms.every(function (t) { return hay.indexOf(t) >= 0; });
    });
    status.textContent = '找到 ' + hits.length + ' 篇';
    results.innerHTML = hits.length
      ? hits.map(function (p) { return card(p, terms); }).join('')
      : '<p class="empty-hint">没有匹配的文章，换个关键词试试。</p>';
  }

  fetch('{{ "/search.json" | relative_url }}')
    .then(function (r) { return r.json(); })
    .then(function (data) {
      all = data;
      var q = new URLSearchParams(window.location.search).get('q');
      if (q) input.value = q;
      run(input.value);
    })
    .catch(function () {
      status.textContent = '索引加载失败，请刷新重试。';
    });

  var timer;
  input.addEventListener('input', function () {
    clearTimeout(timer);
    var v = input.value;
    timer = setTimeout(function () { run(v); }, 120);
  });
})();
</script>
