require 'rubygems'
require 'rake'
require 'fileutils'

desc "Draft a new post"
task :new do
  puts "What should we call this post for now?"
  name = STDIN.gets.chomp
  FileUtils.touch("drafts/#{name}.md")

  open("drafts/#{name}.md", 'a') do |f|
    f.puts "---"
    f.puts "comments: true"
    f.puts "date: 0000-00-00 15:00:00"
    f.puts "layout: post"
    f.puts "slug: some-dash-delimited-phrase"
    f.puts "title: My Title" 
    f.puts "summary: \"Summary\""
    f.puts "image: 'post-subfolder/image.png'"
    f.puts "tags:"
    f.puts "- atagname"
    f.puts "---"
  end
end


desc "Startup Jekyll"
task :start do
  sh "jekyll server --watch --port 4001"
end

task :default => :start

