# Contributing

This doc explains how to get started contributing to [Baikonur](https://github.com/baikonur-oss) project.

## Filing an Issue

If you are trying to use Baikonur modules and run into some issues, please file an issue. We can look into the issue more easily if you describe in details as possible as you can. The following is example detailed descriptions:

- Version of tools and modules
- Expected and actual results
- Error messages with debug option
- Steps to reporoduce the issues

## Submitting a PR

PRs are welcome! Submitting an issue and discussing is a good practice, so feel free to open new issues. Baikonur project is young, and we still have to figure out guidelines etc., so submitting an issue prior to submitting a PR might be better.

## Releasing

The release process is as follows:

<!--1. Make sure the direction for addressing the issues under discussion-->
1. Accept a PR and review immpementation
1. Additional E2E test or something using codes within the branch
1. Get approval and merge it into master
1. Create a new incremented release from releases page and upload Lambda package named `lambda_package.zip`

The release process for new module goes something like this:

1. Review first implementation including sample code pointing to the Terraform Registry URI
1. Put v0.0.1 tag on **PR branch**
1. Register module with Terraform Registry
1. Check the appearance of modules in Terraform Registry
1. Additional E2E, unit tests (with additional v0.0.x releases if needed)
1. Get approval and merge it into master
1. Put v1.0.0 tag on **master branch**
1. Create an initial release from release page and upload Lambda package named `lambda_package.zip`
1. Delete pre-release tags (v0.0.1, v0.0.2, ...)
