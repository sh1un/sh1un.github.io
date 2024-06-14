# sh1un.github.io

This repository contains the source code for my [personal blog](https://shiun.me), built with Hugo and the PaperMod theme.

## Structure

The repository is structured as follows:

- `.github/`: Contains GitHub Actions workflows for deploying the site.
- `blog/`: Contains the Hugo source files for the blog.
  - `archetypes/`: Contains the archetype files for new content.
  - `assets/`: Contains web assets like the site manifest.
  - `content/`: Contains the Markdown files for the blog posts.
  - `layouts/`: Contains the layout files for the site.
  - `public/`: Contains the generated static site files.
  - `themes/`: Contains the Hugo themes, including PaperMod.
- `CNAME`: Contains the custom domain for the GitHub Pages site.
- `README.md`: This file.

## Building

The site is built and deployed automatically using the [`Deploy Hugo PaperMod Demo to Pages`](../../../c:/GitHub/sh1un.github.io/.github/workflows/hugo.yaml) GitHub Actions workflow.

## Reporting Issues

Feel free to report any issues with the blog by [creating a new issue](https://github.com/sh1un/sh1un.github.io/issues/new)
