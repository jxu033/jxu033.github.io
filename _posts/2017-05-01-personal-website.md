---
title: "Jiaqi's blog"
layout: post
date: 2017-05-01
tag:
- jekyll
- markdown

projects: true
hidden: true # don't count this post in blog pagination
description: "A online store"
jemoji: '<img class="emoji" title=":jekyll:" alt=":jekyll:" src="/assets/images/language_icon/jekyll.png" height="20" width="20" align="absmiddle">'
author: Jiaqi Xu
externalLink: false
---

Jekyll is a Ruby Gem, and can be installed on most systems.

#### Environment Setting
* Install [Jekyll](http://jekyllrb.com), [NodeJS](https://nodejs.org/) and [Bundler](http://bundler.io/).
* Clone the forked repo on your machine.
* cd to the repo directory and run  `bundle install`, it will install all the packages in Gemfile.
* Then run `bundle exec jekyll serve --config _config.yml,_config-dev.yml`
* Open it in your browser: `http://localhost:4000`
* Test your app with `bundle exec htmlproofer ./_site`

You must fill some informations on `_config.yml` to customize your site.

#### Pacakge.json
This file is used to indicate that this project is a node project.

```text
"scripts": {
  "build": "bundle exec jekyll build --config _config.yml,_config_dev.yml",
  "serve": "bundle exec jekyll serve --config _config.yml,_config_dev.yml",
  "test": "bundle exec htmlproofer ./_site"
}
```

#### Simplify the command to run locally
run  `npm run serve`

#### Test
You can test your app with: run `npm run test`

#### Link
[Jiaqi's blog](http://jqx.world/)
