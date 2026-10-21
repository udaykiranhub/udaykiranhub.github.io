source "https://rubygems.org"

# Only needed to preview the site locally. GitHub Pages builds the site
# itself with this same gem, so local and live builds match.
gem "github-pages", group: :jekyll_plugins

# Ruby 3+ no longer bundles webrick, which `jekyll serve` needs.
gem "webrick"

# Windows has no system time zone database.
platforms :windows, :jruby do
  gem "tzinfo-data"
end
