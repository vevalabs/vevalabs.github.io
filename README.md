# VevaLabs Website

This is source code repository for [VevaLabs](https://vevalabs.dev) website for VevaLabs at Boğaziçi University. Built with [Hugo](https://gohugo.io/) using the [Hextra](https://github.com/imfing/hextra) theme, and deployed to GitHub Pages.

## Prerequisites

- [Hugo](https://gohugo.io/installation/) (extended edition)
- [Go](https://go.dev/dl/) 1.21+

## Development

Run the local dev server with live reload:

```sh
hugo server
```

The site will be available at `http://localhost:1313/`.

## Build

```sh
hugo
```

Output is written to the `public/` directory.

## Deployment

The site is automatically built and deployed to GitHub Pages via the workflow in [.github/workflows/pages.yaml](.github/workflows/pages.yaml) on pushes to `main`.

## Content

Site content lives under [content/](content/), written in Markdown. Configuration is in [hugo.yaml](hugo.yaml).
