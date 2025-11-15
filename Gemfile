source "https://rubygems.org"

# GitHub Pages gem - includes Jekyll and all allowed plugins
gem "github-pages", group: :jekyll_plugins

# Ruby 3.4 compatibility - these are no longer bundled
gem "erb"
gem "logger"
gem "csv"
gem "base64"
gem "bigdecimal"

# Optional: Windows and JRuby compatibility
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Performance booster for watching directories on Windows
gem "wdm", "~> 0.1", platforms: [:mingw, :x64_mingw, :mswin]

# Lock http_parser.rb to v0.6.x for JRuby
gem "http_parser.rb", "~> 0.6.0", platforms: [:jruby]
