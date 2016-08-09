# fadeitdk-blog

##Install:
```bash
gem install jekyll bundler
bundle install
npm install
```

Run dev server at http://localhost:4000/blog:
```bash
jekyll serve
```

Build site (`_site/`):
```bash
jekyll build
```


##Tips on design
When setting text/meta colors, check [all colors for reference](https://www.materialui.co/colors).

You can set the sizeDesktop (desktop size, duh) of a certain blog in the bloglist (home page or /blog). It's a 12-column grid system, recommended sizes are 4, 6, 12 (depending on length / importance of the post). 

See defaults in _config.yml._
