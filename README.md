Jekyll is a Ruby Gem, and can be installed on most systems.

#### Environment Setting
* Install [Jekyll](http://jekyllrb.com), [NodeJS](https://nodejs.org/) and [Bundler](http://bundler.io/).
* Clone the forked repo on machine.
* cd to the repo directory and run  `bundle install`, it will install all the packages in Gemfile.lock.
* Then run `bundle exec jekyll serve --config _config.yml,_config-dev.yml`
* Open it in browser: `http://localhost:4000`
* Test app with `bundle exec htmlproofer ./_site`

Now, we can some informations on `_config.yml` to customize our site.<br>
We can also write blog, project pages using Markdown and design our website.

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
We can test your app with: run `npm run test`

#### Link
[Jiaqi's blog](http://xjq.world/)
