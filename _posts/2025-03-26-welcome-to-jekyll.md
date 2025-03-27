---
layout: post
title:  "Steps to build a free tech blog on Github"
date:   2025-03-26 20:27:29 +0800
categories: jekyll update front-end
---

Inspired by blog [Sell yourself, Sell Your Work][sell-urs-sell-ur-work], a software engineer should leverage his/her own work project achievement to sell himself, instead of solving tickets piece by piece alone. I decided to created my own blog.

Considering there is a wide range of choice in the market, most are in charge service fee so that less effort to maintain a blog service under the ground, I, instead, prefer a little bit tech but fun way to build my blog with no charge.

<!-- describe my initiatie and goal of blog -->

Some blogs and posts guide me to a balanced solution: Github Page + Jekyll. I log here for someone to refer to.

Firstly, I need to create a public repo named raypan-wq.github.io, the prefix name must matchs your github username.

Then clone and commit a dummy index.html to test. After pushing, you'd wait for second to let github serve your static website. Then visit `https://raypan-wq.github.io/` to validate if works.

```sh
git clone blog-url
cd blog
echo "hello word" > index.html
```

Good, next step is to install static web generator on local, which is written in ruby requiring version >= 3, so try install ruby version management tool first.

```sh
brew install chruby ruby-install
ruby-install ruby 3.4.1
```

```sh
echo "source $(brew --prefix)/opt/chruby/share/chruby/chruby.sh" >> ~/.zshrc
echo "source $(brew --prefix)/opt/chruby/share/chruby/auto.sh" >> ~/.zshrc
echo "chruby ruby-3.4.1" >> ~/.zshrc 
source ~/.zshrc
```
validate ruby version with `ruby -v` ensure greate than 3, then install core binaries.

```sh
gem install jekyll bundler
```

Vola! Now we can create a local blog

```sh
jekyll new docs
cd ./docs
bundle exec jekyll serve --livereload
```

You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

[sell-urs-sell-ur-work]: https://www.solipsys.co.uk/new/SellYourselfSellYourWork.html?yc25hn