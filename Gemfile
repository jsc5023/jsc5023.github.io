# frozen_string_literal: true

source "https://rubygems.org"

gem "jekyll", "~> 4.3.0"
gem "jekyll-theme-chirpy", "~> 7.2.0"

group :test do
  gem "html-proofer", "~> 4.4"
end

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Lock `http_parser.rb` gem to `v0.6.x` on JRuby builds since newer versions of the gem
# do not have a Java counterpart.
gem "http_parser.rb", "~> 0.6.0", platforms: :jruby

gem 'wdm', '>= 0.1.0' if Gem.win_platform?

gem 'logger'
gem 'csv'
gem 'ostruct'
gem 'base64'

# 플러그인 추가
gem "jekyll-feed", "~> 0.15.0"
gem "jekyll-seo-tag", "~> 2.8.0"
gem "jekyll-sitemap", "~> 1.4.0"
