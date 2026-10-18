source "https://rubygems.org"

# GitHub Pages 会用它自己的构建环境编译本站，
# 这个 Gemfile 的主要用途是：本地预览保持和线上行为一致。
#
# 本地预览：
#   bundle install
#   bundle exec jekyll serve --livereload --drafts
#
# 线上（GitHub Pages）用的是它锁定的一套 gem 版本（jekyll 3.10 / minima 2.5.1）。
# 本地因此也按同一套版本约束，避免「本地能跑、线上构建失败」。
gem "github-pages", group: :jekyll_plugins

group :jekyll_plugins do
  gem "jekyll-feed"
end

# Windows / JRuby 上的时区数据
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", "~> 1.2"
  gem "tzinfo-data"
end

# Windows 上的文件监听加速
gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]

# Ruby 3+ 不再内置 webrick
gem "webrick", "~> 1.8"
