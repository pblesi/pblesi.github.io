source "https://rubygems.org"
ruby "3.4.4"

# Hello! This is where you manage which Jekyll version is used to run.
# When you want to use a different version, change it below, save the
# file and run `bundle install`. Run Jekyll with `bundle exec`, like so:
#
#     bundle exec jekyll serve
#
# This will help ensure the proper Jekyll version is running.
# Happy Jekylling!
#gem "jekyll", "3.4.3"

# If you want to use GitHub Pages, remove the "gem "jekyll"" above and
# uncomment the line below. To upgrade, run `bundle update github-pages`.
require 'json'
require 'open-uri'
versions = JSON.parse(URI.open('https://pages.github.com/versions.json').read)
gem "github-pages", versions['github-pages'], group: :jekyll_plugins

# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.6"
  gem "jekyll-paginate", "~> 1.1.0"
  gem "jekyll-gist", "~> 1.5.0"
  gem "jekyll-sitemap", "~> 1.0"
  gem "jekyll-seo-tag", "~> 2.5"
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
