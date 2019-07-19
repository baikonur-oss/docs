# Contributing

This doc explains how to get started contributing to [baikonur-oss](https://github.com/baikonur-oss) project.

## Filing an Issue

If you are trying to use baikonur-oss modules and run into some issues, please file an issue. We can look into the issue more easily if you describe in details as possible as you can. The following is example detailed descriptions:

- Version of tools and modules
- Expected and actual results
- Error messages with debug option
- Steps to reporoduce the issues

## Submitting a PR

Before submitting a PR, make sure that there's an issue filed for your work. It might need some discussion before addressing the issue. Filing a issue first will help us to save your time to solving misunderstandings between you and us.

## Releasing

The release process for existing module goes something like this:

1. Make sure the direction for addressing the issues under discussion
1. Accept a PR and review implementation
1. Additional E2E test or something using codes within the branch
1. Get approval and merge it into master
1. Increce version number on **master branch**

The release process for new module goes something like this:

1. Review first implementation including sample code pointing to the Terraform Registry URI
1. Put v0.0.1 tag on **PR branch**
1. Register module with Terraform Registry
1. Check the appearance of modules in Terraform Registry
1. Additional E2E test or something using codes tagged with v0.1.0 if needed
1. Get approval and merge it into master
1. Put v1.0.0 tag on **master branch**
1. Delete pre-release tags (v0.0.1, v0.0.2, ...)
