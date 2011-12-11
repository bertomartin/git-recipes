require 'yaml'
require 'digest'
require 'builder'
require 'rake/clean'

begin
  require 'kramdown'
rescue LoadError
  abort 'Dependency error: gem kramdown is required. Install with `gem install kramdown`.'
end

CLEAN.include 'book', 'preview.html'
CLOBBER.include '*.epub'

CONFIG    = YAML.load_file('config.yml')
STRUCTURE = %w[book/META-INF/container.xml book/mimetype book/OEBPS/styles.css book/OEBPS/toc.ncx book/OEBPS/content.opf]

## Classes ##############################################################

module Epub
  class XmlFile
    attr_reader :xml

    def initialize
      @xml = Builder::XmlMarkup.new indent: 2
    end

    def to_s
      render
      xml.target!
    end
  end

  class Container < XmlFile
    def render
      xml.instruct!
      xml.container version: '1.0', xmlns: 'urn:oasis:names:tc:opendocument:xmlns:container' do
        xml.rootfiles do
          xml.rootfile 'full-path' => 'OEBPS/content.opf', 'media-type' => 'application/oebps-package+xml'
        end
      end
    end
  end

  class Content < XmlFile
    attr_reader :book

    def initialize(book)
      @book = book
      super()
    end

    def render
      xml.instruct!
      xml.declare! :DOCTYPE, :package, :PUBLIC,  '-//W3C//DTD XHTML 1.1//EN', 'http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd'
      xml.package 'xmlns' => 'http://www.idpf.org/2007/opf', 'unique-identifier' => 'BookId', 'version' => '2.0' do
        xml.metadata 'xmlns:dc' => 'http://purl.org/dc/elements/1.1/', 'xmlns:opf' => 'http://www.idpf.org/2007/opf' do
          xml.dc :language,    @book.language
          xml.dc :title,       @book.title
          xml.dc :creator,     @book.creator, 'opf:role' => 'aut'
          xml.dc :publisher,   @book.publisher
          xml.dc :subject,     @book.subject
          xml.dc :identifier,  @book.uid, id: 'BookId'
          xml.dc :rights,      @book.rights
          xml.dc :description, @book.description
        end

        xml.manifest do
          xml.item 'media-type' => 'application/x-dtbncx+xml', 'href' => 'toc.ncx', 'id' => 'ncx'
          xml.item 'media-type' => 'text/css', 'href' => 'styles.css', 'id' => 'css'
          @book.chapters.each do |chapter|
            xml.item 'id' => chapter.id, 'href' => chapter.target, 'media-type' => 'application/xhtml+xml'
          end
        end

        xml.spine 'toc' => 'ncx' do
          @book.chapters.each do |chapter|
            xml.itemref 'idref' => chapter.id
          end
        end
      end
    end
  end

  class Toc < XmlFile
    attr_reader :book

    def initialize(book)
      @book = book
      super()
    end

    def render
      xml.instruct!
      xml.declare! :DOCTYPE, :ncx, :PUBLIC, "-//W3C//DTD XHTML 1.1//EN", 'http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd'
      xml.ncx xmlns: 'http://www.daisy.org/z3986/2005/ncx/' , version: '2005-1' do
        xml.head do
          xml.meta name: 'dtb:uid', content: @book.uid
          xml.meta name: 'dtb:depth', content: '1'
          xml.meta name: 'dtb:totalPageCount', content: '0'
          xml.meta name: 'dtb:maxPageNumber', content: '0'
        end
        xml.docTitle { xml.text @book.title }
        xml.navMap do
          @book.chapters.each_with_index do |chapter, n|
            xml.navPoint id: chapter.id, playOrder: n do
              xml.navLabel { xml.text chapter.title }
              xml.content src: chapter.target
            end
          end
        end
      end
    end
  end

  class Book
    include Enumerable
    
    attr_reader :settings, :chapters
    
    def initialize(settings = {})
      @settings = settings
      @chapters = []
    end

    def output_filename
      "#{filename}-#{version}.epub"
    end
    
    def method_missing(name, *args)
      return super unless respond_to? name
      @settings.fetch(name.to_s)
    end
    
    def respond_to?(name)
      super or @settings.has_key? name.to_s
    end
    
    def <<(chapter)
      @chapters << chapter
      @chapters.sort!
      self
    end
    
    def sources
      map(&:source)
    end
    
    def targets
      map(&:target)
    end
    
    def uid
      Digest::SHA1.hexdigest [@settings.inspect, *map(&:uid)].join('--')
    end
    
    def each
      chapters.each { |c| yield c }
    end
    
    def images
      @images ||= map(&:images).flatten.uniq
    end

    def files
      images + sources
    end
  end

  class Chapter
    include Comparable
    
    attr_reader :source
    
    def initialize(filename)
      @source = filename
      @content = File.read(source)
    end
    
    def title
      @title ||= @content[/^# (.+)$/, 1]
    end
    
    def index
      @index ||= source.sub(/^content\/0+/, '').to_i
    end
    
    def id
      @id ||= "chapter_#{index}"
    end
    
    def target
      @target ||= source.sub(/md$/, 'xhtml').sub(/^content\//, '')
    end
    
    def <=>(other)
      index <=> other.index
    end
    
    def uid
      @uid ||= Digest::SHA1.hexdigest [id, @content].join
    end
    
    def images
      @images ||= begin
        source.scan(/!\[([^\]]*)\]\(([^)]+)\)/).map { |m| m[2] }.uniq
      end
    end
  end
end

## Setup and helpers #####################################################

@book = Epub::Book.new(CONFIG)
FileList['content/*.md'].each { |f| @book << Epub::Chapter.new(f) }

INPUT_FILES = FileList.new { |f| f.include(*@book.files) }
OUTPUT_FILES = FileList.new { |f| f.include(*@book.files) }.gsub(/^content/, 'book/OEBPS').gsub(/md$/, 'xhtml')

def find_md(filename)
  INPUT_FILES.find { |s| File.basename(s, 'md') == File.basename(filename, 'xhtml')  }
end

def find_png(filename)
  INPUT_FILES.find(filename)
end

def kramdown(str)
  Kramdown::Document.new(str, auto_ids: false, coderay_line_numbers: nil, template: 'layout.xhtml').to_html
end

## Tasks ################################################################

task default: :compile

desc "Generate #{@book.output_filename}"
task compile: @book.output_filename

desc "Force re-generate #{@book.output_filename}"
task recompile: [:clobber, :compile]

if `which epubcheck` != ''
  desc "Test #{@book.output_filename} with epubcheck"
  task test: @book.output_filename do
    sh "epubcheck", @book.output_filename
  end
else
  desc "Test #{@book.output_filename} with epubcheck"
  task :test do
    warn 'The `epubcheck` program is not available. Make sure it is installed and available in your $PATH.'
  end
end

## Rules ################################################################

rule '.xhtml' => lambda { |file| find_md(file) } do |t|
  Rake::Task['book/OEBPS'].invoke
  File.open(t.name, 'w') do |f|
    f.write kramdown(File.read(t.source))
  end
end

rule '.png' => lambda { |file| find_png(file) } do |t|
  Rake::Task['book/OEBPS'].invoke
  cp t.source t.name 
end


## Directories ###########################################################

directory 'book'
directory 'book/OEBPS'
directory 'book/META-INF'

## Files ################################################################

file 'preview.html' => INPUT_FILES do |t|
  File.open(t.name, 'w') do |f|
    f.write kramdown(t.prerequisites.sort.map { |f| File.read(f) }.join("\n\n"))
  end
end

file 'book/OEBPS/content.opf' => INPUT_FILES.include('book/OEBPS') do |t|
  File.open(t.name, 'w') { |f| f.write(Epub::Content.new(@book)) }
end

file 'book/OEBPS/toc.ncx' => INPUT_FILES.include('book/OEBPS') do |t|
  File.open(t.name, 'w') { |f| f.write(Epub::Toc.new(@book)) }
end

file 'book/mimetype' => ['book'] do |t|
  File.open(t.name, 'w') { |f| f.write('application/epub+zip') }
end

file 'book/META-INF/container.xml' => ['book/META-INF'] do |t|
  File.open(t.name, 'w') { |f| f.write(Epub::Container.new) }
end

file 'book/OEBPS/styles.css' => ['styles.css'] do |t|
  Rake::Task['book/OEBPS'].invoke
  cp 'styles.css', t.name
end

file 'book/book.zip' => OUTPUT_FILES.include(*STRUCTURE) do |t|
  Dir.chdir 'book' do
    sh 'zip', '-0X', 'book.zip', 'mimetype'
    sh 'zip', '-9rg', 'book.zip', 'META-INF', '-x', '\*.DS_Store'
    sh 'zip', '-9rg', 'book.zip', 'OEBPS', '-x', '\*.DS_Store'
  end
end

file @book.output_filename => 'book/book.zip' do |t|
  cp 'book/book.zip', t.name
end
