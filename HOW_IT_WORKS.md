# How This Codebase Works

This is a personal blog and tools site hosted on **GitHub Pages**. It uses
**Jekyll**, a static site generator, to turn simple text files into a complete
website. You write content in HTML or Markdown, push to GitHub, and the site
updates automatically.

---

## What is Jekyll?

Jekyll is a program (written in Ruby) that takes your source files -- HTML
templates, Markdown posts, CSS -- and combines them into a finished website
made entirely of plain HTML/CSS/JS files. There is no database, no server-side
code, and no login. The output is a folder called `_site/` containing the
final website that any browser can open.

GitHub Pages runs Jekyll for you automatically every time you push to the
`master` branch. You never need to run Jekyll yourself (though you can for
local previewing -- see the bottom of this doc).

---

## File tree

```
.
├── _config.yml                              # Site-wide settings for Jekyll
├── Gemfile                                  # Ruby dependencies for local dev
├── .gitignore                               # Files Git should ignore
│
├── _layouts/                                # HTML wrappers (templates)
│   ├── default.html                         #   Base shell used by every page
│   └── post.html                            #   Blog post wrapper (extends default)
│
├── _posts/                                  # Blog posts (Markdown files)
│   └── 2019-12-19-site-working.md           #   One blog post
│
├── blog/
│   └── index.html                           # Blog listing page (shows all posts)
│
├── tools/
│   ├── index.html                           # Tools listing page (like blog index)
│   └── sample-size-calculator/
│       └── index.html                       # Interactive power analysis tool
│
├── css/
│   └── main.css                             # Global stylesheet for the whole site
│
├── images/                                  # Static image assets
├── index.html                               # Homepage
├── HOW_IT_WORKS.md                          # This file (excluded from the site build)
└── README.md
```

---

## The big picture: how a page gets built

When Jekyll builds your site, it follows this pipeline for every page:

```
 1. Read the file (e.g. index.html)
 2. See the "front matter" block (the --- section at the top)
 3. Process any Liquid template tags (like {{ page.title }})
 4. If it's Markdown, convert it to HTML
 5. Wrap it in the layout specified in front matter
 6. If that layout has its own layout, wrap again (layout chaining)
 7. Write the final HTML to _site/
```

**Example**: a blog post goes through this chain:

```
 _posts/2019-12-19-site-working.md       (your Markdown content)
         ↓  rendered into HTML
 _layouts/post.html                      (adds title + date header)
         ↓  wrapped inside
 _layouts/default.html                   (adds nav, footer, CSS link)
         ↓  written to
 _site/blog/2019/12/19/site-working.html (final page visitors see)
```

---

## Key concepts explained

### Front matter

The block of `---` lines at the top of a file. This is YAML metadata that
tells Jekyll how to process the file. Every file that Jekyll should process
needs front matter (even if it's empty).

```yaml
---
layout: default
title: My Page Title
---
```

- **layout** -- which HTML wrapper to use (from `_layouts/`)
- **title** -- the page title, accessible as `{{ page.title }}` in templates

If a file has no front matter, Jekyll copies it to `_site/` as-is without
processing.

### Liquid template tags

Jekyll uses a templating language called Liquid. You'll see two kinds of tags:

- **Output tags** `{{ ... }}` -- print a value. Example: `{{ page.title }}`
  prints the page's title.
- **Logic tags** `{% raw %}{% ... %}{% endraw %}` -- control flow. Example: `{% raw %}{% for post in site.posts %}{% endraw %}`
  loops through all blog posts.

These only work inside files that have front matter.

### Layouts (`_layouts/`)

Layouts are HTML wrappers. Think of them like picture frames -- your content
goes inside.

**`default.html`** is the base frame for every page on the site:

```
┌─────────────────────────────────────┐
│  <head> title, CSS link </head>     │
│                                     │
│  <nav> Home | Blog | Tools </nav>   │
│                                     │
│  ┌───────────────────────────────┐  │
│  │                               │  │
│  │   {{ content }}               │  │
│  │   (your page goes here)      │  │
│  │                               │  │
│  └───────────────────────────────┘  │
│                                     │
│  <footer> email link </footer>      │
└─────────────────────────────────────┘
```

This is why every page on the site has the same nav bar and footer -- they
all use this layout. If you want to change the nav links (e.g., add a new
section), you edit `_layouts/default.html`.

**`post.html`** adds a title and date above blog post content, then wraps
itself inside `default.html`. This is called **layout chaining** -- one
layout can extend another.

### Pages

Any file with front matter becomes a page. The URL is determined by its
location in the file tree:

| File | URL | What it does |
|------|-----|--------------|
| `index.html` | `/` | Homepage with greeting |
| `blog/index.html` | `/blog` | Lists all blog posts |
| `tools/index.html` | `/tools` | Lists all tools |
| `tools/sample-size-calculator/index.html` | `/tools/sample-size-calculator/` | The interactive calculator |

### Blog posts (`_posts/`)

Posts have a special naming convention: `YYYY-MM-DD-slug.md`

- The date is parsed from the filename
- The slug becomes part of the URL
- The permalink pattern in `_config.yml` controls the final URL structure

So `2019-12-19-site-working.md` becomes `/blog/2019/12/19/site-working`.

Posts automatically show up in the blog listing page -- you don't need to
manually add links. The blog listing page loops through all posts using
Liquid.

---

## Configuration (`_config.yml`)

```yaml
name: Archi's Blog
url: "https://archisman-roy.github.io"
baseurl: ""
markdown: kramdown
permalink: /blog/:year/:month/:day/:title
exclude:
  - HOW_IT_WORKS.md
  - Gemfile
  - Gemfile.lock
```

| Setting | What it does |
|---------|-------------|
| `name` | Your site's name (available as `{{ site.name }}` in templates) |
| `url` | The full URL where the site is hosted |
| `baseurl` | Path prefix if the site lives in a subdirectory (empty for user sites) |
| `markdown: kramdown` | The Markdown engine Jekyll uses |
| `permalink` | URL pattern for blog posts (`:year`, `:month`, `:day`, `:title` are placeholders) |
| `exclude` | Files that Jekyll should NOT process or copy to `_site/` |

The `exclude` list is important -- without it, Jekyll tries to process
`HOW_IT_WORKS.md` and fails because it contains Liquid code examples that
look like real template tags.

---

## How the stylesheet works (`css/main.css`)

One CSS file styles the entire site:

- **Body**: centered at 70% width with 60px top/bottom margin
- **Nav & footer**: horizontal lists, Helvetica, bold, no bullets
- **Links**: gray (#999), underline on hover
- **Headings**: 3em, Helvetica
- **Paragraphs**: 1.5em, dark gray
- **Code blocks**: monospace font, light gray background
- **Post list** (`ul.posts`): also used by the tools listing page

Individual pages (like the calculator) add their own `<style>` blocks at the
top of the file. These styles only affect that page and override the global
CSS where needed, without touching `main.css`.

---

## How the Sample Size Calculator works

`tools/sample-size-calculator/index.html`

This is a fully client-side interactive tool -- no server or backend needed.
It uses the same `default` layout as every other page, then adds page-specific
CSS and JavaScript inline.

### Layout (side-by-side)

```
┌─────────────────────┬───────────────────────────────────┐
│  Controls Panel     │                                   │
│  ┌───────────────┐  │      D3.js Chart                  │
│  │ n       [==■=]│  │                                   │
│  │ σ       [==■=]│  │   ╭─H₀─╮        ╭─H₁─╮          │
│  │ δ       [==■=]│  │  ╱      ╲  ←δ→  ╱      ╲         │
│  │ α       [==■=]│  │ ╱   α    ╲____╱  power  ╲        │
│  └───────────────┘  │                                   │
│                     │      [legend]                     │
│  ┌─Power──────────┐ │                                   │
│  │    85.3%       │ ├───────────────────────────────────┤
│  ├─Required n─────┤ │  Stacks vertically on mobile     │
│  │    175         │ │                                   │
│  └────────────────┘ │                                   │
└─────────────────────┴───────────────────────────────────┘
```

Controls sit on the left so you can drag sliders and see the chart update
in real time without scrolling.

### The math (pure JavaScript, no dependencies)

Three helper functions implement normal distribution math:

| Function | What it computes |
|----------|-----------------|
| `pdf(x, mu, s)` | The height of the bell curve at point x |
| `cdf(x)` | The area under the standard normal curve up to x |
| `qnorm(p)` | The x-value where the area under the curve equals p |

These feed two calculations:

**Statistical power** (probability of detecting a real effect):
```
SE    = σ × sqrt(2/n)             ← how much noise is in your measurement
z_c   = qnorm(1 - α/2)           ← the critical threshold for significance
power = 1 - cdf(z_c - δ/SE)      ← chance the alternative clears the threshold
      + cdf(-z_c - δ/SE)         ← (left tail, usually negligible)
```

**Required sample size** (for 80% power):
```
n = ceil( 2 × σ² × (z_α/2 + z_0.8)² / δ² )
```

### The visualization (D3.js)

D3 v7 is loaded from a CDN (`d3js.org`). The chart is an SVG element that
scales responsively using `viewBox`.

Every time a slider changes, the `render()` function:

1. Recomputes SE and the critical value
2. Generates 300 data points each for the null and alternative distributions
3. Updates SVG path elements (layered back-to-front):
   - **Rejection areas** (light red) -- where we'd reject the null hypothesis
   - **Power areas** (light green) -- where the alternative distribution falls
     in the rejection region
   - **Null curve** (blue) -- distribution assuming no effect
   - **Alternative curve** (red) -- distribution assuming the true effect = δ
4. Updates critical value lines, labels, and the MDE annotation (δ bracket)
5. Rescales axes to fit the current distributions

### How the interactivity works

```
   Slider dragged
        │
        ▼
   Update state object S = { n, sigma, mde, alpha }
        │
        ├──▶ Sync the number input to match slider value
        │
        └──▶ Call render()
                │
                ├──▶ Recompute power & required n → update display cards
                │
                └──▶ Recompute & redraw all SVG elements
```

Each slider and number input are wired bidirectionally -- changing either
one updates the shared state and triggers a full re-render.

---

## How GitHub Pages deploys the site

```
 You run: git push origin master
           │
           ▼
 GitHub detects the push
           │
           ▼
 GitHub Pages runs Jekyll (its own version, not yours)
           │
           ▼
 Jekyll builds _site/ from your source files
           │
           ▼
 _site/ is served at https://archisman-roy.github.io
```

GitHub Pages uses its own Ruby environment and Jekyll version (currently 3.10).
Your Gemfile is for local development only.

---

## Files Git ignores (`.gitignore`)

```
_site/          # Jekyll's build output -- GitHub rebuilds this on every push
Gemfile.lock    # Exact gem versions -- not needed since GitHub uses its own
```

You never commit build output or lock files for this project.

---

## Local development

To preview the site on your computer before pushing:

```bash
# First time: make sure Homebrew Ruby is in your PATH
export PATH="/opt/homebrew/opt/ruby/bin:/opt/homebrew/lib/ruby/gems/4.0.0/bin:$PATH"

# First time: install dependencies
bundle install

# Start the local server
bundle exec jekyll serve
```

Then open `http://localhost:4000`. The server watches for file changes and
auto-rebuilds (except `_config.yml` changes, which need a restart).

**Note**: the Gemfile uses Jekyll 4 for local development because the
`github-pages` gem (which pins Jekyll 3.9) isn't compatible with modern
Ruby (4.x). This is fine -- your site is simple and builds identically
on both versions.

---

## How to add new content

### New blog post

1. Create a file in `_posts/` named `YYYY-MM-DD-your-title.md`
2. Add front matter at the top:
   ```yaml
   ---
   layout: post
   title: "Your Post Title"
   date: YYYY-MM-DD
   ---
   ```
3. Write your content in Markdown below the front matter
4. Push to GitHub -- it automatically appears on the `/blog` page

### New standalone page

1. Create `your-page/index.html` (or `your-page.html`)
2. Add front matter with `layout: default` and a `title`
3. Optionally add a nav link in `_layouts/default.html`

### New interactive tool

1. Create `tools/your-tool/index.html` with `layout: default`
2. Add `<style>` for page-specific CSS and `<script>` for JavaScript
3. Load any libraries from CDN (e.g., D3.js, Chart.js)
4. Add a list entry in `tools/index.html` so it shows up on the tools page
5. Everything runs client-side in the browser -- no backend needed
