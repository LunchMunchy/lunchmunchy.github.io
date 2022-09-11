---
layout: post
title: Why I skipped Day 1 of 100DaysOfKubernetes
subtitle: A soul crushing experience
tags: [100DaysOfKubernetes, Programming]
comments: true
readtime: true
---

# 🤨 What went wrong?

I was supposed to start my crazy K8 adventure. What stopped me? Not knowing how to set up a blog. If setting up the right dev environment makes-or-breaks a coder, then setting up the right notes environment makes-or-breaks a learner

This morning I spent 5 hours bashing my head against the wall that is my keyboard - metaphysically. Here's a the series of questions I asked to slowly figure out my way to victory.

1. What is Jekyll and why do bloggers love it?
1. How do you incorporate it with GitHub Pages?
1. "How to setup ~~Minima~~ ~~Hyde~~ ~~Chirpy~~" 
1. How come this [insert issue] isn't working???
1. "What is Jekyll made easy"
1. Will I be able to make my #100DaysOfCode post?
1. What will my 3 non-coder Twitter followers think?
1. "How to setup Beautiful Jekyll"
1. ~~Why didn't I use this sooner?~~ Why am I stupid?

Eventually I got my blog working and a twitter post out (1 hour before midnight). This means tomorrow I'll hop on the [#100DayOfKubernetes](https://100daysofkubernetes.io/) train, hopefully🤞.

# 💡 Here's what I learned
If Wordpress is the blogger-platform-posterboy, then Jekyll is the "I'm a hipster coder" version of Wordpress.

I hate Ruby. But not as much as I used to.

GitHub pages is really not that hard. All you need to do is create a repository and name it `[Insert Anything].github.io`. Then have an ```index.html``` or something that renders a webpage (like a Jekyll theme). That's it. 90 whole minutes of tinkering just to come to that conclusion. 💀

Once I got Beautiful Jekyll working, I decided to write my post. How Jekyll Works is you need:
1. Create a [post](https://jekyllrb.com/docs/posts/) with a ```YEAR-MONTH-DAY-title.md``` title
2. Add a [YAML Front Matter](https://jekyllrb.com/docs/posts/) block to start of the file
3. Let your creative juices flow freely on the pages
4. Allow your chosen Jekyll theme to magically format your ```.md```

Because I wanted to automate step 2, I created a script ***heavily*** borrowed from [Kostas Karampinas](https://rpk.io/posts/automatically-create-jekyll-posts-with-thor) and [Chad](https://gist.github.com/ichadhr/0b4e35174c7e90c0b31b).

```ruby
require "stringex"
require "thor"

class Post < Thor
  desc "new TITLE", "Create a new post"

  method_option :date, aliases: "-d",
                       default: Time.now.strftime('%Y-%m-%d'),
                       desc: "Change the current time to the value provided"
  method_option :category, aliases: "-c",
                           default: "General",
                           type: :string,
                           desc: "Add the post's category, default 'General'"
  method_option :tags, aliases: "-t",
                       type: :string,
                       desc: "Add post tags, comma-separated string"
  method_option :editor, :default => "code"

  def new(*title)
    title = title.join(" ")
    category = options[:category]
    # fomat tags in the way you want them to be displayed
    tags = options[:tags].gsub(/\W+/, ", ") if options[:tags]
    #filename = "_posts/#{category.downcase}/#{options[:date]}-#{title.to_url}.md"
    filename = "_posts/#{options[:date]}-#{title.to_url}.md"

    if File.exist?(filename)
      abort("#{filename} already exists!")
    end

    puts "Creating new post: #{filename}"
    open(filename, 'w') do |post|
      post.puts "---"
      post.puts "layout: post"
      post.puts "title: #{title.gsub(/&/,'&amp;')}"
      post.puts "subtitle: [REPLACE ME]"
      post.puts "tags: \[#{tags}\]"
      post.puts "cover-img: [REPLACE_ME]"       # Unnecessary
      post.puts "thumbnail-img: [REPLACE ME]"   # Unnecessary
      post.puts "---"
    end

    # opens the md file in your default editor
    # system ("#{ENV['EDITOR']} #{filename}")  <<< only works if VSC already open
    system(options[:editor], filename)

    puts "New post created: #{filename}"
  end
end
```

I recommend you read their posts if you want to automatically create jekyll posts yourself. But as long as you
```
gem install thor
gem install stringex
```
then you can copy and paste my code and it should work. Here's the code I used to create this post:
```
thor post:new Why I skipped Day 1 of 100DaysOfKubernetes -c DevSecOps -t "Programming,#100DaysOfKubernetes"
```
Finally, I think I'm going to migrate from Beautiful Jekyll to something else downt he road. Maybe be a pro coder and make my own. If I hate myself.

# 💤 TL;DR

Here's my poem summarizing it all

*After all of this I feel I might be slightly dumb. \
But at least today is done, another day will come.*