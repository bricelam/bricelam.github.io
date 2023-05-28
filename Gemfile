source 'https://rubygems.org'

gem 'jekyll', '~> 3.9.3'
gem 'minima', '~> 2.0'

group :jekyll_plugins do
  gem 'jekyll-redirect-from', '0.16.0'
  gem 'jekyll-sitemap', '1.4.0'
  gem 'jekyll-paginate', '1.1.0'
  gem 'jemoji', '0.12.0'
  gem 'jekyll-mentions', '1.6.0'
end

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem 'tzinfo', '>= 1', '< 3'
  gem 'tzinfo-data'
end

# Performance-booster for watching directories on Windows
gem 'wdm', '~> 0.1.0', :install_if => Gem.win_platform?

gem 'kramdown-parser-gfm'

# Lock `http_parser.rb` gem to `v0.6.x` on JRuby builds since newer versions of the gem
# do not have a Java counterpart.
gem 'http_parser.rb', '~> 0.6.0', :platforms => [:jruby]
