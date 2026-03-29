# How This Codebase Works

This is a **Jekyll-powered static site** hosted on GitHub Pages at
`https://archisman-roy.github.io`. Jekyll reads the source files in this repo,
processes Liquid templates and Markdown, and outputs a complete static website
into a `_site/` folder (which is `.gitignore`-d).

---

## File tree

```
.
├── _config.yml                          # Jekyll site-wide settings
├── Gemfile                              # Ruby dependencies (github-pages gem)
├── _layouts/
│   ├── default.html                     # Base HTML shell (every page uses this)
│   └── post.html                        # Blog-post wrapper (extends default)
├── _posts/
│   └── 2019-12-19-site-working.md       # A blog post (Markdown)
├── blog/
│   └── index.html                       # Blog listing page
├── css/
│   └── main.css                         # Global stylesheet
├── images/                              # Static image assets
├── tools/
│   └── sample-size-calculator/
│       └── index.html                   # Interactive power-analysis tool
├── index.html                           # Homepage
├── .gitignore                           # Ignores _site/
└── README.md
```

---

## How Jekyll builds the site

### 1. Configuration (`_config.yml`)

```yaml
name: Archi's Blog
url: "https://archisman-roy.github.io"
baseurl: ""
markdown: kramdown
permalink: /blog/:year/:month/:day/:title
```

- **name** / **url** / **baseurl** -- identify the site; used by Liquid tags
  like `{{ site.url }}`.
- **markdown: kramdown** -- the Markdown engine Jekyll uses to convert `.md`
  files into HTML.
- **permalink** -- controls the URL pattern for blog posts. A post dated
  `2019-12-19` with slug `site-working` becomes
  `/blog/2019/12/19/site-working`.

### 2. Gemfile

```ruby
source "https://rubygems.org"
gem "github-pages", group: :jekyll_plugins
```

Tells GitHub Pages which Jekyll version and plugins to use when building the
site. Without this file GitHub Pages may fail to build.

### 3. Layouts (`_layouts/`)

Layouts are HTML wrappers. A page specifies its layout in **front matter**
(the `---` YAML block at the top of a file).

#### `default.html` -- the base shell

```
<!DOCTYPE html>
<html>
  <head>
    <title>{{ page.title }}</title>
    <link rel="stylesheet" href="/css/main.css">
  </head>
  <body>
    <nav>  Home | Blog | Tools  </nav>
    <div class="container">
      {{ content }}          <-- page body gets injected here
    </div>
    <footer> email link </footer>
  </body>
</html>
```

Every page on the site goes through this layout. The `{{ content }}` tag is
where Jekyll injects the calling page's rendered body.

#### `post.html` -- blog post layout

```
---
layout: default          <-- inherits from default.html
---
<h1>{{ page.title }}</h1>
<p class="meta">{{ page.date | date_to_string }}</p>
<div class="post">
  {{ content }}
</div>
```

This layout **extends** `default.html` (note `layout: default` in its front
matter). Jekyll first renders the post Markdown into HTML, injects it into
`post.html` at `{{ content }}`, then injects that result into `default.html`
at its own `{{ content }}`. This is layout chaining.

### 4. Pages

Any `.html` or `.md` file with front matter (`---` block) becomes a page.

| File | URL | What it does |
|------|-----|--------------|
| `index.html` | `/` | Homepage -- shows a greeting and blurb |
| `blog/index.html` | `/blog` | Lists all posts with `{% for post in site.posts %}` |
| `tools/sample-size-calculator/index.html` | `/tools/sample-size-calculator/` | Interactive JS tool (see below) |

### 5. Blog posts (`_posts/`)

Files in `_posts/` follow the naming convention `YYYY-MM-DD-slug.md`. Jekyll
parses the date and slug from the filename. Each post declares front matter:

```yaml
---
layout: post
title: "Well, This is working"
date: 2019-12-19
---
```

Jekyll converts the Markdown body to HTML, wraps it in `post.html`, which
wraps it in `default.html`. The post shows up in the `site.posts` collection,
which `blog/index.html` iterates over.

---

## How the stylesheet works (`css/main.css`)

A single CSS file styles the entire site:

- **Body**: centered at 70% width with 60px top/bottom margin.
- **Nav & footer**: horizontal `<ul>` lists, Helvetica, bold, no bullets.
- **Links**: gray (`#999`), underline on hover.
- **Headings**: 3em Helvetica.
- **Paragraphs**: 1.5em with `#333` color.
- **Code blocks** (`pre`/`code`): monospace font, light gray background.
- **Post list** (`ul.posts`): 1.5em, no bullets.

Individual pages (like the calculator) add page-scoped `<style>` blocks to
override or extend these base styles without modifying `main.css`.

---

## How the Sample Size Calculator works

`tools/sample-size-calculator/index.html`

This is a fully client-side interactive tool -- no server needed. It uses the
`default` layout like any other page, then adds its own `<style>` block and
JavaScript.

### Structure

```
┌─────────────────────────────────────────────┐
│  <style> ... </style>   (page-scoped CSS)   │
│                                             │
│  <div class="calculator">                   │
│    ├── Controls panel (4 sliders + inputs)  │
│    ├── Results panel  (power + required n)  │
│    └── Chart container (D3 SVG + legend)    │
│  </div>                                     │
│                                             │
│  <script src="d3.v7.min.js">  (CDN)         │
│  <script> ... </script>  (calculator logic) │
└─────────────────────────────────────────────┘
```

### Math (inside the `<script>`)

Three core math functions, all pure JavaScript with no dependencies:

| Function | What it computes |
|----------|-----------------|
| `pdf(x, mu, s)` | Normal probability density function |
| `cdf(x)` | Standard normal CDF (Abramowitz & Stegun approximation) |
| `qnorm(p)` | Inverse normal / quantile function (rational approximation) |

These feed two statistical calculations:

**Power** (given n, sigma, MDE, alpha):
```
SE    = sigma * sqrt(2/n)           # standard error of mean difference
z_c   = qnorm(1 - alpha/2)         # critical z-value
power = 1 - cdf(z_c - MDE/SE)      # right-tail detection
      + cdf(-z_c - MDE/SE)         # left-tail detection (usually tiny)
```

**Required sample size** (for 80% power):
```
n = ceil( 2 * sigma^2 * (z_{alpha/2} + z_{0.8})^2 / MDE^2 )
```

### Visualization (D3.js)

The chart is an SVG rendered by D3 v7 (loaded from CDN). On every slider
change the `render()` function:

1. Recomputes `SE` and the critical value `cv = z_c * SE`.
2. Generates 300 data points each for the null `N(0, SE^2)` and alternative
   `N(MDE, SE^2)` distributions.
3. Sets x/y scales to fit both curves with some padding.
4. Updates six SVG `<path>` elements (layered back-to-front):
   - **Rejection areas** (light red) -- area under null curve beyond +/-cv.
   - **Power areas** (light green) -- area under alternative curve beyond +/-cv.
   - **Null curve** (blue line) -- the full null distribution.
   - **Alternative curve** (red line) -- the full alternative distribution.
5. Updates dashed critical-value lines and axis labels.

The SVG uses `viewBox` for responsive scaling -- it automatically fits any
container width.

### Wiring (slider <-> state <-> chart)

A shared state object `S = { n, sigma, mde, alpha }` is the single source of
truth. Each slider and number input listens for `input` events, updates `S`,
syncs its counterpart (slider <-> input), and calls `render()`. This creates
the real-time interactive feel.

---

## How GitHub Pages deploys the site

1. You push to the `master` branch.
2. GitHub Pages detects the `Gemfile` and runs Jekyll to build `_site/`.
3. The contents of `_site/` are served at `https://archisman-roy.github.io`.

The `_site/` directory is in `.gitignore` because GitHub rebuilds it on every
push -- you never need to commit build output.

---

## How to add new content

**New blog post**: create `_posts/YYYY-MM-DD-your-slug.md` with front matter
`layout: post`, `title`, and `date`. It automatically appears on `/blog`.

**New standalone page**: create `folder/index.html` (or `page.html`) with
front matter `layout: default` and a `title`. Add a nav link in
`_layouts/default.html` if you want it in the menu.

**New interactive tool**: same as a standalone page, but add `<style>` and
`<script>` blocks for CSS/JS. Load libraries from CDN. Everything runs
client-side.
