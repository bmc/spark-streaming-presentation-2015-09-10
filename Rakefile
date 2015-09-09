# -*- ruby -*-

require 'tempfile'
require 'grizzled/fileutil/includer'
require 'nokogiri'
include Grizzled::FileUtil

# ----------------------------------------------------------------------------
# Constants
# ----------------------------------------------------------------------------

OUTPUT_DIR     = "dist"
HTML_SOURCES   = Dir.glob("*.html") + Dir.glob("slides/**/*.html")
OUTPUT_SLIDES  = "#{OUTPUT_DIR}/index.html"
SLIDES_LIST    = "slide-list.tmp"
# Since SVG images are inlined, we don't want to copy them. We also
# don't want to copy iDraw sources.
IMAGES_TO_COPY = Dir.glob("images/*.{png,jpg}")

# ----------------------------------------------------------------------------
# Tasks
# ----------------------------------------------------------------------------

task :default => :build
task :build => :html
task :dist => :build
task :html => OUTPUT_SLIDES

file OUTPUT_SLIDES => [:css, :js, :images] + HTML_SOURCES do
  puts "presentation.html => #{OUTPUT_DIR}/index.html"
  inc = Includer.new("presentation.html",
                     include_pattern: '^\s*%include\s"([^"]+)',
                     allow_glob: true)
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

task :images => [:svg, "images/"] do
  mkdir_p "#{OUTPUT_DIR}/images"
  IMAGES_TO_COPY.each { |p| cp p, "#{OUTPUT_DIR}/images" }
end

SVG_FILES = Dir.glob("images/**/*.svg")
task :svg => SVG_FILES do
  # For each file, modify each <g> element with an "id" attribute that
  # starts with "Layer" so that the <g> element is a Reveal.js fragment.
  SVG_FILES.each do |svg|
    puts "Animating layers in #{svg}..."
    augment_svg svg
  end
end

task :css => ["#{OUTPUT_DIR}/css/custom.css"]

file "#{OUTPUT_DIR}/css/custom.css" => ['custom.less'] do
  mkdir_p "#{OUTPUT_DIR}/css"
  sh 'sh', '-c', "lessc custom.less >#{OUTPUT_DIR}/css/custom.css"
end

task :clean do
  rm_rf OUTPUT_DIR
end

# ----------------------------------------------------------------------------
# Helper functions
# ----------------------------------------------------------------------------

# Augment an SVG in two ways:
#
# 1. Remove the "x", "y" and "viewBox" attributes from the <svg> element.
# 2. Find any layers and mark them with class="fragment", so that Reveal.js
#    will animated them. A layer is assumed to be an SVG group (<g> element)
#    with an ID. Tools like iDraw use <g> elements with "id" attributes to
#    mark layers.
#
# Parameters:
#
# svg_file           - the path to the SVG image
# add_fragment_index - If true (the default), also add a
#                      data-fragment-index="n" attribute to each layer <g>
#                      element, to force the layers to show up in the order
#                      they appear in the image.
def augment_svg(svg_file, add_fragment_index: true)
  def get_classes(element)
    attr = element.attribute('class')
    attr = attr.value.split(/\s+/) if attr
    attr
  end

  def set_classes(element, array)
    element['class'] = array.join(" ")
  end

  image = File.open(svg_file) { |f| Nokogiri::XML(f) }
  image.remove_namespaces!
  layers = image.xpath("//g[@id]")
  if layers.length == 0
    # Nothing to do
  elsif layers.length == 1
    # There's only one layer. No sense causing incremental display.
    # Remove any existing fragments.
    g = layers[0]
    classes = Set.new(get_classes(g))
    if classes
      classes.delete('fragment')
      set_classes(g, classes.to_a)
    end
  else
    layers.each_with_index do |g, i|
      classes = get_classes(g)
      if classes
        classes << 'fragment' unless classes.member?('fragment')
      else
        classes = ['fragment']
      end
      set_classes(g, classes)

      if add_fragment_index
        g['data-fragment-index'] = (i + 1).to_s
      end
    end
  end

  %w{x y viewBox}.each { |attr| image.root.delete(attr) }

  File.open svg_file, "w" do |f|
    f.write(image.to_xml)
  end
end
