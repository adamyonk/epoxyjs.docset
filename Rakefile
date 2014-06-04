require 'nokogiri'

# File structure
SOURCE = 'source'
DOCSET = 'epoxyjs.docset'
CONTENTS = "#{DOCSET}/Contents"
RESOURCES = "#{CONTENTS}/Resources"
DOCUMENTS = "#{RESOURCES}/Documents"
PLIST = "#{CONTENTS}/Info.plist"
INDEX = "#{RESOURCES}/docSet.dsidx"

task default: [:build]

directory DOCUMENTS

directory SOURCE

file "#{SOURCE}/backbone.epoxy.tgz" => [SOURCE] do |t|
  sh "curl --location https://github.com/gmac/backbone.epoxy/archive/master.tar.gz -o #{t.name}"
end

file PLIST => [DOCUMENTS] do |t|
  sh "curl --location http://kapeli.com/dash_resources/Info.plist -o #{t.name}"
  text = File.read(t.name).gsub(/nginx/, 'epoxyjs').gsub(/Nginx/, 'Epoxy.js')
  File.open(t.name, 'w') do |file|
    file.puts text
  end
end

directory "#{SOURCE}/backbone.epoxy-master" => ["#{SOURCE}/backbone.epoxy.tgz"] do |t|
  mkdir t.name
  sh "tar -xzf #{t.prerequisites[0]} -C #{SOURCE}"
end

file "#{DOCUMENTS}/documentation.html" => ["#{SOURCE}/backbone.epoxy-master", DOCUMENTS] do |t|
  puts t.prerequisites[0]
  cp_r Dir["#{t.prerequisites[0]}/www/*"], DOCUMENTS
end

file INDEX => ["#{DOCUMENTS}/documentation.html"] do |t|
  sh "sqlite3 #{t.name} \"CREATE TABLE searchIndex(id INTEGER PRIMARY KEY, name TEXT, type TEXT, path TEXT);\""
  sh "sqlite3 #{t.name} \"CREATE UNIQUE INDEX anchor ON searchIndex (name, type, path);\""
  # TODO: add getting started guides
  html = Nokogiri::HTML(File.open(t.prerequisites[0], 'r'))
  html.css('h3').each do |el|
    sh "sqlite3 #{t.name} \"INSERT INTO searchIndex VALUES (NULL, '#{el.text}', 'Function', 'documentation.html##{el.attr('id')}');\""
  end
end

task :clean do |t|
  rm_rf [DOCSET, SOURCE, "#{DOCSET}.tgz"]
end

desc "(default) Downloads the latest Epoxy.js docs and builds a new docset"
task :build => [PLIST, INDEX] do |t|
  sh "tar --exclude='.DS_Store' -cvzf #{DOCSET}.tgz #{DOCSET}"
end

