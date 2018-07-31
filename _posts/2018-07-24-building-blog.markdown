---
title: building-blog
layout: post
guid: urn:uuid:1b7c8cda-f1a7-45a1-80d4-d56a6fcb6996
tags:
  - 
---


### install latest version of ruby by rvm [RVM](https://rvm.io/rvm/install)

1. `brew install gnupg`; [Homebrew](http://brew.sh/)
2. `gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB`; 请参考官方[RVM](https://rvm.io/rvm/install)
3. `\curl -sSL https://get.rvm.io | bash -s stable --ruby`; **\**的作用是使用原本的curl命令，而不是被重新定义(Alias)的，具体参考[stackoverflow](https://stackoverflow.com/questions/15951772/curl-bash-whats-the-slash-for)
4. 畅快的安装完成后，重新打开终端就可以了

```
$ rvm list
=* ruby-2.5.1 [ x86_64 ]

# => - current
# =* - current && default
#  * - default

$ rvm which ruby
/Users/xxx/.rvm/rubies/ruby-2.5.1/bin/ruby
$ rvm which gem
/Users/xxx/.rvm/rubies/ruby-2.5.1/bin/gem

```

-------------------

到这里，ruby应该是最新版本啦，接下来安装`jekyll`, 使用`jekyll`创建博客。参考[官方文档](https://jekyllrb.com/docs/installation)

`gem install bundler jekyll`

创建博客: `jekyll new jekyll-website` 或者：

$ `mkdir jekyll-blog`

$ `cd jekyll-blog`

\# Create a Gemfile

$ `bundle init`

\# Add Jekyll

$ `bundle add jekyll`

\# Install gems

$ `bundle install`
