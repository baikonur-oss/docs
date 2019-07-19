<p align="center">
  <br>
  <img width="800" src="./images/logo_l_black@4x.png" alt="logo of baikonur/docs repository">
  <br>
  <br>
</p>


# Baikonur Project Homepage

Welcome to Baikonur!

Baikonur is an open source [Terraform Modules](https://www.terraform.io/docs/modules/usage.html) project by [CyberAgent, Inc.](https://www.cyberagent.co.jp/en/) 

Baikonur started as an internal project in 2018, and quickly grew to over 50 modules.
We are releasing select most popular and unique modules in this organization.

Baikonur project name comes from [Baikonur Cosmodrome](https://en.wikipedia.org/wiki/Baikonur_Cosmodrome), 
a spaceport located in Kazakhstan known for launching the first artificial satellite and the first human spaceflight.

Modules released here are also published to Terraform Public Module Registry: [Baikonur top page](https://registry.terraform.io/modules/baikonur-oss).

## Usage
Follow instructions in repository READMEs or [Terraform Module Registry pages](https://registry.terraform.io/modules/baikonur-oss).
Refer to examples in repositories, and don't forget to use proper [versioning](https://www.terraform.io/docs/configuration/modules.html#module-versions).

## Modules

- AWS
  - [iam-nofile](https://github.com/baikonur-oss/terraform-aws-iam-nofile) - module to make it easier to create AWS IAM Roles. You can write role policy document with inline(heredoc) syntax, so you do not have to use [template rendering](https://www.terraform.io/docs/providers/template/d/file.html) in order to use variables.

  - Modules for logging with Kinesis and Lambda
    - [lambda-kinesis-to-fluent](https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-to-fluent) - Kinesis Data Streams -> Fluent transfer
    - [lambda-kinesis-to-s3](https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-to_s3) - Kinesis Data Streams -> S3 transfer
    - [lambda-kinesis-to-es](https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-to_es) - Kinesis Data Streams -> Elasticsearch transfer
    - [lambda-es-cleaner](https://github.com/baikonur-oss/terraform-aws-lambda-es_cleaner) - Old indices cleaner for Elasticsearch Service
    - [lambda-kinesis-forward](https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-forward) - Kinesis Data Streams -> Kinesis Data Streams forwarder/router module

- GCP
  - coming soon


## Contributing
PRs, issues, and any other form of feedback is very welcome in all repositories under baikonur-oss organization! :smile:

If you have a module you would like to donate to Baikonur, let us know in Issues of this repository.

<!-- Maintainers and contributors wanted! -->

## License

All repositories in this organization are provided on MIT license (see LICENSE file in repositories). 

Baikonur logos are CC0.
