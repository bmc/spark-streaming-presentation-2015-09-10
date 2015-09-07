# -*- ruby -*-

require 'tempfile'
require 'grizzled/fileutil/includer'
include Grizzled::FileUtil

task :default => :build

OUTPUT_DIR    = "dist"
HTML_SOURCES  = Dir.glob("*.html") + Dir.glob("slides/**/*.html")
OUTPUT_SLIDES = "#{OUTPUT_DIR}/index.html"

task :build => :html

task :html => OUTPUT_SLIDES

file OUTPUT_SLIDES => [:css, :js, :images] + HTML_SOURCES do
  puts "presentation.html => #{OUTPUT_DIR}/index.html"
  inc = Includer.new("presentation.html", include_pattern: '^\s*%include\s"([^"]+)')
  lines = inc.to_a
  File.open "#{OUTPUT_DIR}/index.html", "w" do |f|
    lines.each do |line|
      f.write(line)
    end
  end
end

task :js => ["bower_components/reveal.js",
             "bower_components/jquery",
             "js/highlightjs"] do
  mkdir_p "#{OUTPUT_DIR}/js"
  cp_r "bower_components/reveal.js", "#{OUTPUT_DIR}/js"
  cp_r "bower_components/jquery", "#{OUTPUT_DIR}/js"
  cp_r "js/highlightjs", "dist/js"
end

task :images => ["images/"] do
  cp_r "images", OUTPUT_DIR
end

task :css => ["#{OUTPUT_DIR}/css/custom.css"]

file "#{OUTPUT_DIR}/css/custom.css" => ['custom.less'] do
  mkdir_p "#{OUTPUT_DIR}/css"
  sh 'sh', '-c', "lessc custom.less >#{OUTPUT_DIR}/css/custom.css"
end

task :clean do
  rm_rf OUTPUT_DIR
end
