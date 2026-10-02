# Blog

Jekyll site for GitHub Pages.

## Add a post
Create `_posts/YYYY-MM-DD-your-title.md`:

    ---
    title: "Your post title"
    tags: [endpoint, intel]
    series: endpoint     # optional, groups posts into a series
    part: 2              # optional, order inside the series
    image: /assets/img/banner.png   # optional banner
    ---
    Your Markdown here.

Commit and push. GitHub rebuilds the site in about a minute.

## Run locally (optional)
    bundle install
    bundle exec jekyll serve
