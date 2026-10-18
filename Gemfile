source "https://rubygems.org"

# 本地预览与 GitHub Actions 构建一致。
# 线上构建见 .github/workflows/pages.yml（Jekyll 4 + 本 Gemfile）。
#
#   bundle install
#   bundle exec jekyll serve --livereload
#
gem "jekyll", "~> 4.4"

group :jekyll_plugins do
  gem "jekyll-paginate"
  gem "jekyll-sitemap"
  gem "jekyll-feed"
  gem "jekyll-seo-tag"
  gem "kramdown-parser-gfm"
end

# Windows / JRuby 时区数据
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Windows 文件监听加速
gem "wdm", "~> 0.1", :platforms => [:mingw, :x64_mingw, :mswin]

# Ruby 3+ 不再内置 webrick
gem "webrick", "~> 1.8"
