# Development

This doc describes how to setup a local development environment so you can get started contributing to [baikonur-oss](https://github.com/baikonur-oss) project.

Follow the instructions below to setup your local development environment. Once you complete the following items, you can make changes to baikonur-oss modules.

## Install requirements

You must install the following tools:

- [Terraform](https://www.terraform.io/): Infrastructure as Code tools for automating deployment
  - Currently, all modules are built on top of the Terraform v0.11.x, so you must pick up the latest version from [v0.11 releases](https://releases.hashicorp.com/terraform/)
- [terraform-docs](https://github.com/segmentio/terraform-docs): A utility to generate documentation from Terraform modules
- [pre-commit](https://pre-commit.com/): A framework for running [Git hook](https://git-scm.com/docs/githooks) automatically on every commit

## Checkout the reporitory

To checkout this repository:

1. Create your own [fork of a repository](https://help.github.com/articles/fork-a-repo/)
1. Clone it into your local machine:

```bash
mkdir baikonur-oss
cd baikonur-oss
git clone git@github.com:${YOUR_GITHUB_USERNAME}/${BAIKONUR_MODULE_NAME}.git
cd ${BAIKONUR_MODULE_NAME}
git remote add upstream git@github.com:baikonur-oss/${BAIKONUR_MODULE_NAME}.git
git remote set-url --push upstream no_push
```

We recommend to add the `upstream` remote for regulary [syncing your fork](https://help.github.com/articles/syncing-a-fork/).

## Install pre-commit hooks

Activate the pre-commit hook and have the following benefits before committing the changes:

- Apply terraform format
- Automatically generate inputs/outputs doc sections based on the declarations in variables/outputs files

```bash
pre-commit install
```
