---
layout: post
title: "Create new post with CLI"
tags:
 -
---

Add the following to your Gemfile:

```ruby
gem 'thor'
gem 'stringex'
```

Run bundle install and create a jekyll.thor file with the following contents:

```ruby
require "stringex"
class Jekyll < Thor
  desc "new", "create a new post"
  method_option :editor, :default => "subl"
  def new(*title)
    title = title.join(" ")
    date = Time.now.strftime('%Y-%m-%d')
    filename = "_posts/#{date}-#{title.to_url}.markdown"

    if File.exist?(filename)
      abort("#{filename} already exists!")
    end

    puts "Creating new post: #{filename}"
    open(filename, 'w') do |post|
      post.puts "---"
      post.puts "layout: post"
      post.puts "title: \"#{title.gsub(/&/,'&amp;')}\""
      post.puts "tags:"
      post.puts " -"
      post.puts "---"
    end

    system(options[:editor], filename)
  end
end
```

Use the new command:

`$ thor jekyll:new The title of the new post`