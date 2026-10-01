---
title: Blog Generator
date: 2026-06-28
tags: python, web
vibe: true
---

Custom generator used for creating this blog.

## How it works

I wanted to create blazingly fast, static blog. Write posts in `.md` that I'm used to for making everyday notes.

Blog structure

```
data/
├── posts/
|   ├── 01_inceptor.md
|   └── ...
├── public/
|   ├── img/
|   |   └── ...
|   └── style.css
├── templates/
|   ├── base.html
|   ├── index.html
|   └── post.html
└── posts.xml
```

There are three main directories:
`posts` this is content directory, every post is a separate file
`public` all files from here are just copied to the output
`templates` define the website layout

In addition to that I use `posts.xml` to define which posts are visible and what's the ordering.

```xml
<posts>
  <post file="01_inceptor.md" slug="blog-inceptor" />
</posts>
```

## CLI usage

Having the above structure you just call `build` command to generate a static website.

```bash
python build.py build        # generate dist/
python build.py serve        # local preview at localhost:8000
python build.py new "Title"  # scaffold a new post
python build.py clean        # remove dist/
```

## Code highlighting

I use `Pygments` for static syntax highlighting in different programming languages:

```py
from pathlib import Path

def load_post(slug: str) -> str:
    path = Path("posts") / f"{slug}.md"
    return path.read_text(encoding="utf-8")
```

```go
func greet(name string) string {
    return fmt.Sprintf("Hello, %s!", name)
}
```

```rust
fn main() {
    println!("Hello, world!");
}
```

### CI/CD with Github

To make the publishing process for this website even easier I run the tool with github actions every time there's a commit to repository that contains blog data.

```yml
name: Build and deploy

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout content repo
        uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v5

      - name: Setup Python via uv
        run: |
          uv python install 3.13
          uv venv --python 3.13

      - name: Install blog generator (from separate repo)
        run: |
          uv pip install git+https://github.com/LukeNzk/inceptor.git

      - name: Build site
        run: |
          uv run blog build

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: .dist

  deploy:
    needs: build
    runs-on: ubuntu-latest

    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
