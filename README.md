# Developer Documentation for France-IOI Algorea

Code for auto-generated static website: https://france-ioi.github.io/algorea-devdoc/

## Quick Start

The doc is auto-generated by Github pages at every push on master. If you want to run it locally:

1. Install the Ruby Development Package: `apt install ruby-dev`
2. Install Ruby and bundler (`gem install bundler`)
3. Install the dependencies: `bundle install`
4. Run it `bundle exec jekyll serve` (or build it once `bundle exec jekyll build`)

### Initialize search data

```
bundle exec just-the-docs rake search:init
```

## Style

For basic style (indent style, line break), make sure your editor support [EditorConfig](https://editorconfig.org/) so that the one defined in this project is used by everybody.

For the text style, try to use markdown file as much as possible. Have a look at [Just-the-docs documentation](https://pmarsceill.github.io/just-the-docs/docs/ui-components) for ways to do custom styling.

### Using navigation and search

Check how to do that with [Just-the-Docs](https://pmarsceill.github.io/just-the-docs/docs/navigation-structure/)
