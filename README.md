# panvimdoc

Write documentation in [pandoc markdown](https://pandoc.org/MANUAL.html).
Generate documentation in vimdoc.

# Motivation

Writing documentation is hard.
Writing documentation in vimdoc for vim plugins is an additional hassle.
Making writing vim plugin documentation frictionless is important.

Writing documentation in markdown and converting it to vimdoc can help toward that goal.
This way, plugin authors will have to write documentation just once ( for example, as part of the README of the project ), and the vim documentation can be autogenerated.

Writing vim documentation requires conforming to a few simple rules.
Although `vimdoc` is not a well defined spec, vim does have some nice syntax highlighting and features like tags and links when the text file is in `vimdoc` compatible format and when `filetype=help` in vim.
Also, typically, while vim documentation are just plain text files, they are usually formatted well using whitespace.
See <https://vimhelp.org/helphelp.txt.html#help-writing> or [`@nanotree`'s project](https://github.com/nanotee/vimdoc-notes) for more information.
I think preserving these features and characteristics of vimdoc for documentation of vim plugins is important.

Writing documentation in Markdown and converting it to vimdoc is not a novel idea.
[`@mjlbach`](https://github.com/mjlbach) has already implemented a neovim treesitter based markdown to vimdoc converter that works fairly well.
See [mjlbach/babelfish.nvim](https://github.com/mjlbach/babelfish.nvim) for more information.
This approach is close to ideal. There are no dependencies ( except for the Markdown treesitter parser ). While it appears that the markdown parser may cause crashes, I have not experienced any issues in my use. It is neovim only but you can use this on github actions even for a vim plugin documentation.

I found two other projects that do something similar, [wincent/docvim](https://github.com/wincent/docvim) and [FooSoft/md2vim](https://github.com/FooSoft/md2vim).
As far as I can tell, these projects are actively maintained and may suit your need.

However, none of these projects use Pandoc.
Pandoc Markdown supports a wide number of features: See <https://pandoc.org/MANUAL.html> for more information.
Most importantly, it supports a range of Markdown formats and flavors.
And, Pandoc has filters and a custom output writer that can be configured in lua.
Pandoc filters can extend the capability of Pandoc with minimal lua scripting, and these are very easy to write and maintain too.

This project aims to write a specification of syntax in Pandoc Markdown, and to take advantage of Pandoc filters and the custom output writer capability, to convert a Markdown file to a vim documentation help file.
This project provides a reference implementation of the specification as well.

# Goals

- Markdown file must be readable when the file is presented as the README on GitHub / GitLab / SourceHut etc.
- Markdown file converted to HTML using Pandoc must be web friendly and render appropriately (if the user chooses to do so).
- Vim documentation generated must support links and tags.
- Vim documentation generated should be aesthetically pleasing to view, in vim and as a plain text file.
  - This means using columns and spacing appropriately.
- Format of built in Vim documentation is used as guidelines but not as rules.

# Features

- Autogenerate title for vim documentation
- Autogenerate table of contents
- Generate links and tags
- Support markdown syntax for tables
- Support raw vimdoc syntax where ever needed for manual control.
- Support including multiple Markdown files

# Specification

The specification is described in [panvimdoc.md](./doc/panvimdoc.md) along with examples.
The generated output is in [panvimdoc.txt](./doc/panvimdoc.txt).
The reference implementation of the Pandoc lua filter is in [panvimdoc.lua](./scripts/panvimdoc.lua).
See [entrypoint.sh](./entrypoint.sh) for how to use this script, or check the [Usage](#usage) section.

If you would like to contribute to the specification, have feature requests or opinions, please feel free to comment on this issue to start a discussion: <https://github.com/kdheepak/panvimdoc/issues/1>.

# Usage

```bash
pandoc --metadata=project:${PROJECT} --lua-filter scripts/skip-blocks.lua --lua-filter scripts/include-files.lua -t scripts/panvimdoc.lua ${INPUT} -o ${OUTPUT}
```

The following are the metadata fields that the custom writer uses:

- `project` (String) _required_: This is typically the plugin name. This is prefixed to all generated tags. (e.g. `*project-heading*`)
- `toc` (Boolean) _optional_: Whether to generate table of contents or not. If not present, this value is set to `true`.
- `description` (String) _optional_: The description for your plugin. If not present, the `vimversion` and current date is used.
- `vimversion` (String) _optional_: The version vim / neovim that the plugin is targeting. If not present, the version of vim in the available environment is used.

Example:

```markdown
---
project: panvimdoc
vimversion: Neovim v0.5.0
toc: true
---
```

Generates the following:

```
*panvimdoc.txt*         For Neovim v0.5.0          Last change: 2021 August 12

==============================================================================
Table of Contents                                *panvimdoc-table-of-contents*

1. panvimdoc                                             |panvimdoc-panvimdoc|
2. Motivation                                           |panvimdoc-motivation|
3. Goals                                                     |panvimdoc-goals|
...
```

Adding the `description`:

```markdown
---
project: panvimdoc
description: pandoc markdown to vimdoc
toc: true
---
```

generates the following:

```
*panvimdoc.txt*                                      pandoc markdown to vimdoc
```

## Using Github Actions

Add the following to `./.github/workflows/pandocvim.yml`:

```yaml
name: panvimdoc

on: [push]

jobs:
  docs:
    runs-on: ubuntu-latest
    name: pandoc to vimdoc
    steps:
      - uses: actions/checkout@v2
      - name: panvimdoc
        uses: kdheepak/panvimdoc@main
        with:
          vimdoc: VIMDOC_PROJECT_NAME
          # the following are defaults on github actions
          # pandoc: "README.md"
          # toc: true
          # description: ""
          # version: "NVIM v0.5.0"
      - uses: stefanzweifel/git-auto-commit-action@v4
        with:
          commit_message: "Auto generate docs"
          branch: ${{ github.head_ref }}
```

Choose `VIMDOC_PROJECT_NAME` appropriately.
This is usually the name of the plugin or the documentation file without the `.txt` extension. For example, the following:

```
      - name: panvimdoc
        uses: kdheepak/panvimdoc@main
        with:
          vimdoc: panvimdoc
          description: pandoc markdown to vimdoc
```

Will output a file `doc/panvimdoc.txt` and the vim help tag for it will be `panvimdoc`.
See [`entrypoint.sh`](./entrypoint.sh) for exact shell command executed.

For an example of how this is used, see one of the following workflows:

- [kdheepak/panvimdoc](./.github/workflows/panvimdoc.yml): [doc/panvimdoc.txt](./doc/panvimdoc.txt)
- [kdheepak/tabline.nvim](https://github.com/kdheepak/tabline.nvim/blob/main/.github/workflows/ci.yml): [doc/tabline.txt](https://github.com/kdheepak/tabline.nvim/blob/main/doc/tabline.txt)
- [mcchrish/zenbones.nvim](https://github.com/mcchrish/zenbones.nvim/blob/main/.github/workflows/doc.yml): [doc/zenbones.txt](https://github.com/mcchrish/zenbones.nvim/blob/main/doc/zenbones.txt)

_Feel free to submit a PR to add your documentation as an example here._

# References

- <https://learnvimscriptthehardway.stevelosh.com/chapters/54.html>
- <https://github.com/nanotee/vimdoc-notes>
- <https://github.com/mjlbach/babelfish.nvim>
- <https://foosoft.net/projects/md2vim/>
- <https://github.com/wincent/docvim>
