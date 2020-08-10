---
title: "Creating this site"
date: 2020-06-28T04:14:31-04:00
weight: 1
---

How to build this site

<!--more-->

## Install Hugo

The easiest way is via [prebuilt binaries (github)](https://github.com/gohugoio/hugo/releases)

## Scaffold a site

While it is possible to manually add all the files needed to create the site, the `hugo` tool is very handy when configured correctly for removing a lot of boilerplate.

1. Create a folder for your site (`mkdir my_site && cd my_site`)
2. Run `hugo new site .`


This will a number of different files and folders, but we mostly care about the following:

- `config.toml`: stores the site meta variables like URL and can toggle theme options (described later)
- `layouts`: a folder that will store the HTML templates 
  
    Hugo works similar to other static site generators by taking markdo     wn and metadata and building complete html pages.
    The markdown describes the content, the html describes the layout, and the css will define the style.
- `static`: a folder to store static assets (images etc.)
- `content`: the folder your actual content will be written

## Themes

Themes are packages containing all the html layouts and css for different types of pages for hugo sites.
These make creating a new site super easy, but I have found that configuring them is sometimes not worth the effort for very simple sites.

Most templates seem to assume a blog-style approach where each post is organized / found by a time-based slug.
I prefer to just find my posts by hand-coded urls.

## Working with Templates & Layouts

Layouts stored in the `layouts` folder define the structure of your site.
Different layouts will be automatically loaded based on the context of your site.
The `_default` folder will define the default layout for the following types:

- `baseof.htm`: layout shared by all other layouts
- `single.html`: layout for singles '*posts*'
- `list.html`: layout for pages that list other pages

Below are annotated files that I use to create my base layouts

### Baseof

Inherited by each of the templates.
Each template should include the following:
- class for the site theme
- if the layout should be flipped
- the sidebar
- the actual content

```html
{{ partial "head.html" . }}
  <body class="{{ .Site.Params.themeColor }} {{if .Site.Params.layoutReverse}}layout-reverse{{end}}">
  {{ partial "sidebar.html" . }}
    <main class="content container">
    {{ block "main" . -}}{{- end }}
    </main>
  </body>
</html>
```

### Single

Similarly, this template depicts
- The title
- The date
- table of contents if there is a parameter called `.toc` that is set to the string 'true'
- the actual content

```html
{{ define "main" -}}
<div class="post">
  <h1>{{ .Title }}</h1>
  <time datetime={{ .Date.Format "2006-01-02T15:04:05Z0700" }} class="post-date">{{ .Date.Format "Mon, Jan 2, 2006" }}</time>
  {{ if (eq .Params.toc "true") }}
  <aside>
    {{.TableOfContents}}
  </aside>
  {{ end }}
  {{ .Content }}
</div>

{{- end }}
```

## Syntax Highlighting

Add the following to enable pygments in your config


```toml
# Enable for syntax highlighting
pygmentsUseClasses = true
pygmentsCodefences = true
```

Then, generate a syntax theme with `hugo gen chromastyles --style=github > static/css/syntax.css`.
Finally, load the syntax theme in the `head.html` template

```html
...
  <!-- CSS -->
  <link type="text/css" rel="stylesheet" href="{{ .Site.BaseURL }}css/syntax.css">
...
```