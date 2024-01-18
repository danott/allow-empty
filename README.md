# allow-empty

`allow-empty` is an experimental blog. 
Git commits in this repository are the posts.

## Adding a blog post

```bash
git commit --allow-empty
git push
```

## Development

```bash
git clone git@github.com:danott/allow-empty.git
cd allow-empty
bundle install
bundle exec rake clobber build serve
```

I also reach for [rerun](https://github.com/alexch/rerun) when rapidly iterating.

```bash
# Terminal one
rerun bundle exec rake clobber build -i "_site/*" -p "*"

# Terminal two
bundle exec rake serve

# Terminal three
open "http://localhost:3000"
```

## Deployment

<https://allow-empty.danott.website> is hosted on GitHub pages. 
<https://github.com/danott/allow-empty/blob/main/.github/workflows/github_pages.yml> is the GitHub action that takes care of building and deploying.
Recent deploys can be viewed on GitHub at <https://github.com/danott/allow-empty/deployments> .


## Why

[Just for the fun of it](https://justforfunnoreally.dev).
