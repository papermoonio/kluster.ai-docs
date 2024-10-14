# Documentation for kluster.ai

This repository contains documentation for kluster.ai, the distributed AI platform to efficiently work with large models, free from computational constraints.

## About This Site

This documentation site is generated using [MkDocs](https://www.mkdocs.org). The theme used is [Material for MkDocs](https://squidfunk.github.io/mkdocs-material).

## Run the Site Locally

To get started, you need to install [mkdocs](https://www.mkdocs.org/). All dependencies, including mkdocs, can be installed with a single command; you can run:

```bash
pip install -r requirements.txt
```

In the root directory, run the following command to serve the site locally:

```bash
mkdocs serve
```

After a successful build, the site should be available at `http://127.0.0.1:8000`.

### Edit Theme Files

If you're editing any of the files in the `material-overrides` directory, you can run the following command to watch for these changes and render them automatically:

```bash
mkdocs serve --watch-theme
```

Otherwise, you'll need to stop the server (`control + C`) and restart it (`mkdocs serve`) to see the changes.
