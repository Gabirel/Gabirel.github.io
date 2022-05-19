![preview Long Haul](/preview.jpg)

Long Haul is a minimal jekyll theme built with SASS / GULP / BROWSERSYNC / AUTOPREFIXER and focuses on long form blog posts. It is meant to be used as a starting point for a jekyll blog/website.

This version is modified based on Long Haul.

If you really enjoy Long Haul and want to give me credit somewhere on the internet send or tweet out your experience with Long Haul and tag me [@brianmaierjr](https://twitter.com/brianmaierjr).

#### [View Demo](http://brianmaierjr.com/long-haul)

[![Netlify Status](https://api.netlify.com/api/v1/badges/bd29f13b-3754-46d7-9a39-48db2e174b99/deploy-status)](https://app.netlify.com/sites/long-haul/deploys)

## Features

- Minimal, Type Focused Design
- Built with GULP + SASS + BROWSERSYNC + AUTOPREFIXER
- SVG Social Icons
- Responsive Nav Menu
- XML Feed for RSS Readers
- Contact Form via Formspree
- 5 Post Loop with excerpt on Home Page
- Previous / Next Post Navigation
- Estimated Reading Time for posts
- Stylish Drop Cap on posts
- A Better Type Scale for all devices
- Comments powered by Disqus
- [Dark Mode support](https://github.com/brianmaierjr/long-haul/blob/master/preview-dark.png) via [prefers-color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme) 

## Setup

1. [Install Jekyll](http://jekyllrb.com)
2. Fork the [Long Haul repo](http://github.com/brianmaierjr/long-haul)
3. Clone it
4. [Install Bundler](http://bundler.io/)
5. Run `bundle install`
6. Install gulp dependencies by running `npm install`
7. Run Jekyll and watch files by running `bundle exec gulp`
8. Customize and watch the magic happen!



## Blog Setup

1. ~~Remove all unnecessary files~~

   **Important：You should keep `.git` in order to keep update with upstreams.**

   Skip this steps.

2. Normal setup for jekyll 

   ```shell
   $ bundle update
   $ bundle install
   ```

3. Modify site settings

   - `./_config.yml`
   - `./_data_/config.yml` (recommended)

4. Add your posts

   All existing posts are ok to get removed if you want it.

5. Run preview

   ```shell
   $ jekyll serve -w
   ```


6. Deploy

   ```shell
   # you may change `blog` to something else, or leave it.
   $ git remote add blog <xxxxx>
   
   $ git init && git add . && git commit -m "Add posts"
   
   $ git push --set-upstream blog master
   ```
   
7. Update blogs templates [Optional]

   ```shell
   $ git remote rename origin upstream
   
   # normally you have conflicts, so you need `--rebase`
   $ git pull upstream master --rebase
   ```

   

## Site Settings

The main settings can be found inside the `_config.yml` file:

- **title:** title of your site
- **description:** description of your site
- **url:** your url
- **paginate:** the amount of posts displayed on homepage
- **navigation:** these are the links in the main site navigation
- **social** diverse social media usernames (optional)
- **google_analytics** Google Analytics key (optional)

### Header Option

If you'd like your header to be larger then you can use the option below in you `config.yml` to make it take up half of the vertical space on screens 800px wide and up. *Preview image below.*

- **header:** large

![preview Long Haul](/preview-large.png)

## License

This is [MIT](LICENSE) with no added caveats, so feel free to use this Jekyll theme on your site without linking back to me or using a disclaimer.
