# -*- ruby -*-

require 'tempfile'
require 'grizzled/fileutil/includer'
require 'nokogiri'
include Grizzled::FileUtil

task :default => :build

OUTPUT_DIR    = "dist"
HTML_SOURCES  = Dir.glob("*.html") + Dir.glob("slides/**/*.html")
OUTPUT_SLIDES = "#{OUTPUT_DIR}/index.html"
SLIDES_LIST   = "slide-list.tmp"

task :build => :html
task :dist => :build

task :html => OUTPUT_SLIDES

file OUTPUT_SLIDES => [:css, :js, :images, :slidelist] + HTML_SOURCES do
  puts "presentation.html => #{OUTPUT_DIR}/index.html"
  inc = Includer.new("presentation.html", include_pattern: '^\s*%include\s"([^"]+)')
  lines = inc.to_a
  File.open "#{OUTPUT_DIR}/index.html", "w" do |f|
    lines.each do |line|
      f.write(line)
    end
  end
end

task :slidelist => SLIDES_LIST

file SLIDES_LIST => HTML_SOURCES do
  puts "Creating #{SLIDES_LIST}..."
  # Build slides.tmp, which presentation.html includes.
  File.open SLIDES_LIST, "w" do |f|
    # Note that we only grab the top-level slides. The top-level slides
    # are responsible for pulling in their sub-slides.
    Dir.glob("slides/slide[0-9][0-9].html").each do |path|
      f.write("%include \"#{path}\"\n")
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

task :images => [:svg, "images/"] do
  cp_r "images", OUTPUT_DIR
end

SVG_FILES = Dir.glob("images/**/*.svg")
task :svg => SVG_FILES do
  # For each file, modify each <g> element with an "id" attribute that
  # starts with "Layer" so that the <g> element is a Reveal.js fragment.
  SVG_FILES.each do |svg|
    puts "Animating layers in #{svg}..."
    svg_layers_to_fragments svg
  end
end

task :css => ["#{OUTPUT_DIR}/css/custom.css"]

file "#{OUTPUT_DIR}/css/custom.css" => ['custom.less'] do
  mkdir_p "#{OUTPUT_DIR}/css"
  sh 'sh', '-c', "lessc custom.less >#{OUTPUT_DIR}/css/custom.css"
end

task :clean do
  rm_rf OUTPUT_DIR
  rm_f Dir.glob("*.tmp")
end

def svg_layers_to_fragments(svg_file, add_fragment_index: true)
  image = File.open(svg_file) { |f| Nokogiri::XML(f) }
  image.remove_namespaces!
  image.xpath("//g[contains(@id, 'Layer_')]").each_with_index do |g, i|
    class_attr = g.attributes['class']
    classes = if class_attr
                tokens = class_attr.value.split(/\s+/)
                tokens << 'fragment' unless tokens.member?('fragment')
                tokens
              else
                ['fragment']
              end
    g['class'] = classes.join(" ")
    if add_fragment_index
      g['data-fragment-index'] = (i + 1).to_s
    end
  end

  %w{x y viewBox}.each { |attr| image.root.delete(attr) }

  File.open svg_file, "w" do |f|
    f.write(image.to_xml)
  end
end
