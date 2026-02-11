source "https://rubygems.org"

# Fix SSL CRL error with OpenSSL 3.6.0: use patched openssl gem (see rubyonmac.dev)
gem "openssl", "~> 3.1.2"

# Unpin ffi: override lockfile’s 1.11.1 so we can use a version that builds on Ruby 3.1+
gem "ffi", ">= 1.15.0"

gem "github-pages", group: :jekyll_plugins

gem "tzinfo-data"
gem "webrick"  # Required for `jekyll serve` on Ruby 3.0+ (removed from stdlib)
gem "wdm", "~> 0.1.0" if Gem.win_platform?

# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-paginate"
  gem "jekyll-sitemap"
  gem "jekyll-gist"
  gem "jekyll-feed"
  gem "jemoji"
  gem "jekyll-include-cache"
  gem "jekyll-algolia"
  gem "jekyll-octicons"
end
# Added at 2019-10-09 09:53:47 +0200 by thibault:
gem "pkg-config", "~> 1.3"
