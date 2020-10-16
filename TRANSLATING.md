# Translating

This repository currently is maintained in English and Japanese.

Documentation in this repository includes `.md` files for GitHub, and `.rst` files for our Read the Docs portal at https://baikonur.dev .

## Prerequisites

We use Sphinx for documentation as required by Read the Docs, and Transifex for managing our tranlations.

Make sure to have the following tools installed:
- Sphinx
- sphinx-rtd-theme
- Transifex CLI

Installation instructions for macOS:
```shell
$ pip3 install sphinx sphinx-rtd-theme
$ pip3 install transifex-client
```

Contact one of the Baikonur Organisation owners for a Transifex account and API keys necessary for translations updates.

## Translation flow

```shell
# 1. Generate .po and .pot files from .rst files
$ cd docs
$ sphinx-build -b gettext . _build/gettext

# 2. Push .po files for translation to Transifex
$ tx push -s

# 3. Translate new strings at Transifex

# 4. After translations are finished and reviewed, pull translations from Transifex to local
$ tx pull

# 5. Check if translations are pulled correctly, commit and push changes

# Note: tx pull also creates binary .mo files, make sure to include them as well

# Changes pushed to branches including master will be automatically deployed to baikonur.dev
# Your branch should appear as a new version in version switcher in the bottom-left
```

## For more information

Consult [Read the Docs documentation](https://docs.readthedocs.io/en/stable/guides/manage-translations.html)

