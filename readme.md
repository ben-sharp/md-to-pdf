# Markdown to PDF

![Screenshot of markdown file and resulting PDF](https://file-boswoulruu.now.sh)

**A simple and hackable CLI tool for converting markdown to pdf**. It uses [Marked](https://github.com/markedjs/marked) to convert `markdown` to `html` and [Puppeteer](https://github.com/GoogleChrome/puppeteer) (headless Chromium) to further convert the `html` to `pdf`. It also uses [highlight.js](https://github.com/isagalaev/highlight.js) for code highlighting. The whole source code of this tool is only ~200 lines of JS and ~100 lines of CSS, so it is easy to clone and customize.

**Highlights:**

* Use your own or remote stylesheets
* Front-matter for configuration
* Headers and Footers
* Page Breaks
* Syntax highlighting in code blocks
* Extend the options of the underlying tools

## Installation

```sh
npm i -g md-to-pdf
```

If you want to have your own copy to hack around with, clone the repository instead:

```sh
git clone "https://github.com/simonhaenisch/md-to-pdf"
cd md-to-pdf
npm link # or npm i -g
```

After this, the commands `md-to-pdf` and `md2pdf` (as a shorthand) are globally available in your cli.

## Update

If you already cloned this repository before, you can simply do a `git pull` to pull the latest changes in from the master branch. Unless there have been changes to packages, you don't need to re-install the package (because NPM 5+ uses symlinks, at least on Unix systems).

## Usage

```
$ md-to-pdf [options] <path/to/file.md> [path/to/output.pdf]

Options:

  -h, --help              Output usage information
  -v, --version           Output version
  --stylesheet            Path to a local or remote stylesheet (can be passed multiple times)
  --css                   String of styles (can be used to overwrite stylesheets)
  --body-class            Classes to be added to the body tag (can be passed multiple times)
  --highlight-style       Style to be used by highlight.js (default: github)
  --marked-options        Set custom options for marked (as a JSON string)
  --pdf-options           Set custom options for the generated PDF (as a JSON string)
  --md-file-encoding      Set the file encoding for the markdown file
  --stylesheet-encoding   Set the file encoding for the stylesheet
  --config-file           Path to a JSON or JS configuration file
  --devtools              Open the browser with Devtools instead of saving the PDF (for debugging)
```

The first argument is `path/to/file.md` and the second one optionally specifies the `path/to/output.pdf`. If you omit the second argument, it will derive the pdf name from the markdown filename and save it into the same directory that contains the markdown file. Run `md2pdf --help` for examples on how to use the cli options.

Paths to local images have to be relative to the markdown file location and the files have to be within the same directory the markdown file lives in (or subdirectories of it).

#### Page Break

Place an element with class `page-break` to force a page break at a certain point of the document (uses the CSS rule `page-break-after: always`), e. g.:

```html
<div class="page-break"></div>
```

#### Header/Footer

Set the PDF option `displayHeaderFooter` to `true`, then use `headerTemplate` and `footerTemplate` with the provided classes to inject printing values, e. g. with front-matter (the styles in the `<style/>` tag of the header template will be applied to both header and footer):

```markdown
---
pdf_options:
  format: A4
  margin: 30mm 20mm
  displayHeaderFooter: true
  headerTemplate: |-
    <style>
      section {
        margin: 0 auto;
        font-family: system-ui;
        font-size: 11px;
      }
    </style>
    <section>
      <span class="date"></span>
    </section>
  footerTemplate: |-
    <section>
      <div>
        Page <span class="pageNumber"></span>
        of <span class="totalPages"></span>
      </div>
    </section>
---
```

Refer to the [puppeteer docs](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagepdfoptions) for more info about headers and footers.

#### Default and Advanced Options

For markdown, GFM and tables are enabled by default (see `util/config.js` for default options). The default highlight.js styling for code blocks is `github`.

For advanced options see the following links:

* [Marked Advanced Options](https://github.com/markedjs/marked/blob/master/docs/USING_ADVANCED.md)
* [Puppeteer PDF Options](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagepdfoptions)
* [highlight.js Styles](https://github.com/isagalaev/highlight.js/tree/master/src/styles)

## Options

| Option | Examples |
| - | - |
| `--stylesheet` | `path/to/style.css`, `https://example.org/stylesheet.css` |
| `--css` | `body { color: tomato; }` |
| `--body_class` | `markdown-body` |
| `--highlight-style` | `monokai`, `solarized-light` |
| `--marked-options` | `'{"gfm": false }'` |
| `--pdf-options` | `'{"format": "Letter", margin: "20mm" }'` |
| `--md-file-encoding` | `utf-8`, `windows1252` |
| `--stylesheet-encoding` | `utf-8`, `windows1252` |
| `--config-file` | `path/to/config.json` |
| `--devtools` | opens the browser with devtools (for debugging) |

**`margin`:** instead of an object (as stated in the Puppeteer docs), it is also possible to pass a CSS-like string, e. g. `1em` (all), `1in 2in` (top/bottom right/left), `10mm 20mm 30mm` (top right/left bottom) or `1px 2px 3px 4px` (top right bottom left).

The options can also be set with front-matter or a config file (except `--md-file-encoding` can't be set by front-matter). In that case, remove the leading dashes (`--`) from the cli argument name and replace the hyphens (`-`) with underscores (`_`). `--stylesheet` and `--body-class` can be passed multiple times (i. e. as an array). If the same config option exists in multiple places, the priority (from low to high) is: defaults, front-matter, config file, cli arguments.

Example front-matter:

```markdown
---
stylesheet:
  - path/to/style.css
body_class: markdown-body
highlight_style: monokai
pdf_options:
  format: A5
  margin: 10mm
---

# Content
```

Example `config.json` (can also be a `.js` that default exports an object):

```json
{
  "stylesheet": [
    "path/to/style.css",
    "https://example.org/stylesheet.css"
  ],
  "css": "body { color: tomato; }",
  "body_class": "markdown-body",
  "highlight_style": "monokai",
  "marked_options": {
    "headerIds": false,
    "smartypants": true,
  },
  "pdf_options": {
    "format": "A5",
    "margin": "20mm"
  },
  "stylesheet_encoding": "utf-8"
}
```

#### Github Styles

Here is an example front-matter for how to get Github-like output:

```markdown
---
stylesheet: https://cdnjs.cloudflare.com/ajax/libs/github-markdown-css/2.10.0/github-markdown.min.css
body_class: markdown-body
css: |-
  .page-break { page-break-after: always; }
  .markdown-body { font-size: 11px; }
  .markdown-body pre > code { white-space: pre-wrap; }
---
```

## Customization/Development

You can just start making changes to the files in this repository. NPM 5+ uses symlinks for local global packages, so all changes are reflected immediately without re-installing the package globally (except when there are changes to required packages, then reinstall using `npm i`). This also means that you can just do a `git pull` to get the latest version onto your machine.

Ideas, feature requests and PRs are welcome. Just keep it simple! 🤓

## Credits

Huge thanks to:

* [imcvampire](https://github.com/imcvampire) for handing over the npm package name.
* [Sindre Sorhus](https://github.com/sindresorhus) for amazingly simple to use cli packages.
* [Zeit](https://github.com/zeit) for the inspiration on how to write a cli tool.
* [Josh Bruce](https://github.com/joshbruce) for [reviving Marked](https://github.com/markedjs/marked/issues/1106)
