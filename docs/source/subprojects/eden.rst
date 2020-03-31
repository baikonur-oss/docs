.. _eden_top:

eden: Amazon ECS Dynamic Environment Manager
============================================

Clone Amazon ECS environments easily. Provide eden with a Amazon ECS service and eden will clone it.

eden is fast: create/delete commands should not take more than 5 seconds to run.

Why?
----

eden is made for use cases, where many "preview" development environments are needed, but databases and other resources
may be shared.

How?
----

eden clones Task Definition, ECS Service and Target Group from a reference service.

eden uses one common ALB for all cloned services to save money and time (it can take up to 5 minutes to create an ALB).
eden creates a Target Group, clones a service, attaches created Target Group to common ALB, and creates a
Route 53 A ALIAS record pointing at common ALB.


Resource creation order
^^^^^^^^^^^^^^^^^^^^^^^
1. ECS Task Definition

   - Cloned from reference service

2. ALB Target Group

   - Settings are cloned from the Target Group attached to reference service

3. ECS Service

   - Created in the same cluster as reference service

4. ALB Listener Rule

   - Host Header rule

5. Route 53 A ALIAS record

   - Points at common ALB

6. An entry is added to environments JSON file

.. note::
    Resource deletion is performed in reverse order.
    Both creation and deletion should take no more than 5 seconds.

Prerequisites
-------------

1. Environments JSON file in a S3 bucket

   - Structure and details are described `here <eden_envs_json_>`_

2. A reference ECS Service with Target Group Attached

3. A common ALB for services managed by eden

   - Will be reused by all environments with Host Header Listener Rules
   - Separate from what reference service uses
   - Must have HTTPS Listener
   - Listener must have wildcard certificate for target dynamic zone

4. Simple ALB usage

   - No multiple path rules etc.
   - One ALB per one ECS Service

Preparations
------------

Under construction

..
  TODO: Add instructions on common ALB creation, wildcard certs, dedicated public zone, etc.

.. _eden_envs_json:

Environments JSON file
----------------------

The Environments JSON file is used to:

1. Check what environments exist and where their endpoints are
2. Tell client apps what is available

Environments JSON file example:

.. code-block:: json

    {
        "environments": [
            {
                "env": "dev",
                "name": "dev-dynamic-test",
                "api_endpoint": "api-test.dev.example.com"
            }
        ]
    }

Example above presumes ``config_update_key = api_endpoint``.
You can create environments with the same names, but with profiles with different ``config_update_key`` settings to have
multiple endpoints within a single environment.
For example, you may want to have an API, administration tool, and a frontend service created as a single environment.

Your environment file could look like this:

.. code-block:: json

    {
        "environments": [
            {
                "env": "dev",
                "name": "dev-dynamic-test",
                "api_endpoint": "api-test.dev.example.com",
                "admin_endpoint": "admin-test.dev.example.com",
                "frontend_endpoint": "test.dev.example.com"
            }
        ]
    }

..
  TODO: Show how to create something like above in eden

..
  TODO: Make envs JSON optional

CLI and API
-----------

eden is provided in two flavors: `CLI <aws-eden-cli_>`_ and `API <lambda-eden-api_>`_.

We recommend trying eden out with CLI, and when you feel you are ready to make eden part of your CI/CD pipeline.

Installing eden CLI
^^^^^^^^^^^^^^^^^^^

.. code-block:: console

    $ pip3 install aws-eden-cli

    $ eden -h
    usage: eden [-h] {create,delete,ls,config} ...

    ECS Dynamic Environment Manager. Clone Amazon ECS environments easily.

    positional arguments:
      {create,delete,ls,config}
        create              Create environment or deploy to existent
        delete              Delete environment
        ls                  List existing environments
        config              Configure eden

    optional arguments:
      -h, --help            show this help message and exit

Hint: you can use -h on subcommands as well:

.. code-block:: console

    $ eden config -h
     usage: eden config [-h] {setup,check,push,remote-rm} ...

    positional arguments:
      {setup,check,push,remote-rm}
        setup               Setup profiles for other commands
        check               Check configuration file integrity
        push                Push local profile to DynamoDB for use by eden API
        remote-rm           Delete remote profile from DynamoDB

    optional arguments:
      -h, --help            show this help message and exit

    $ eden config push -h
    usage: eden config push [-h] [-p PROFILE] [-c CONFIG_PATH] [-v]
                            [--remote-table-name REMOTE_TABLE_NAME]

    optional arguments:
      -h, --help            show this help message and exit
      -p PROFILE, --profile PROFILE
                            profile name in eden configuration file
      -c CONFIG_PATH, --config-path CONFIG_PATH
                            eden configuration file path
      -v, --verbose
      --remote-table-name REMOTE_TABLE_NAME
                            profile name in eden configuration file

Configuring eden CLI
^^^^^^^^^^^^^^^^^^^^

Let's create a profile to work with, so we won't have to specify all the parameters every time:

.. code-block:: console

    $ eden config setup --endpoint-s3-bucket-name servicename-config
    $ eden config setup --endpoint-s3-key endpoints.json
    $ eden config setup --endpoint-name-prefix servicename-dev
    $ eden config setup --endpoint-update-key api_endpoint
    $ eden config setup --endpoint-env-type dev
    $ eden config setup --domain-name-prefix api
    $ eden config setup --dynamic-zone-id Zxxxxxxxxxxxx
    $ eden config setup --master-alb-arn arn:aws:elasticloadbalancing:ap-northeast-1:xxxxxxxxxxxx:loadbalancer/app/dev-alb-api-dynamic/xxxxxxxxxx
    $ eden config setup --name-prefix dev-dynamic
    $ eden config setup --reference-service-arn arn:aws:ecs:ap-northeast-1:xxxxxxxxxxxx:service/dev/dev01-api
    $ eden config setup --target-cluster dev

Configuration is saved to ``~/.eden/config``. Commands above created a "default" profile:

.. code-block:: console

    $ cat ~/.eden/config
    [api]
    name_prefix = dev-dynamic
    reference_service_arn = arn:aws:ecs:ap-northeast-1:xxxxxxxxxxxx:service/dev/dev01-api
    target_cluster = dev
    domain_name_prefix = api
    master_alb_arn = arn:aws:elasticloadbalancing:ap-northeast-1:xxxxxxxxxxxx:loadbalancer/app/dev-alb-api-dynamic/xxxxxxxxxx
    dynamic_zone_name = dev.example.com.
    dynamic_zone_id = Zxxxxxxxxxxxx
    config_bucket_name = servicename-config
    config_bucket_key = endpoints.json
    config_update_key = api_endpoint
    config_env_type = dev
    config_name_prefix = servicename-dev
    target_container_name = api

Don't forget to check configuration file integrity:

.. code-block:: console

    $ eden config check
    No errors found

.. _eden_profiles:

Profiles in eden
^^^^^^^^^^^^^^^^

You can create multiple profiles in configuration and specify a profile to use with ``-p profile_name`` for all commands.

.. code-block:: console

    $ eden config check -p api
    No errors found

We can push profiles to DynamoDB for use by eden API:

.. code-block:: console

    $ eden config push -p api
    Waiting for table creation...
    Successfully pushed profile api to DynamoDB

.. note::
    If eden table does not exist, aws-eden-cli will create it

Use the same command to overwrite existing profiles (push to existing profile will result in overwrite):

.. code-block:: console

    $ eden config push -p api
    Successfully pushed profile api to DynamoDB table eden

Use remote-rm to delete remote profiles:

.. code-block:: console

    $ eden config remote-rm -p api
    Successfully removed profile api from DynamoDB table eden

Execute commands
^^^^^^^^^^^^^^^^

Create an environment:

.. code-block:: console

    $ eden create -p api --name foo --image-uri xxxxxxxxxx.dkr.ecr.ap-northeast-1.amazonaws.com/api:latest
    Checking if image xxxxxxxxxx.dkr.ecr.ap-northeast-1.amazonaws.com/api:latest exists
    Image exists
    Retrieved reference service arn:aws:ecs:ap-northeast-1:xxxxxxxxxx:service/dev/api
    Retrieved reference task definition from arn:aws:ecs:ap-northeast-1:xxxxxxxxxx:task-definition/api:20
    Registered new task definition: arn:aws:ecs:ap-northeast-1:xxxxxxxxxx:task-definition/dev-dynamic-api-foo:1
    Registered new task definition: arn:aws:ecs:ap-northeast-1:xxxxxxxxxx:task-definition/dev-dynamic-api-foo:1
    Retrieved reference target group: arn:aws:elasticloadbalancing:ap-northeast-1:xxxxxxxxxx:targetgroup/api/xxxxxxxxxxxx
    Existing target group dev-dynamic-api-foo not found, will create new
    Created target group arn:aws:elasticloadbalancing:ap-northeast-1:xxxxxxxxxx:targetgroup/dev-dynamic-api-foo/xxxxxxxxxxxx
    ELBv2 listener rule for target group arn:aws:elasticloadbalancing:ap-northeast-1:xxxxxxxxxx:targetgroup/dev-dynamic-api-foo/xxxxxxxxxxxx and host api-foo.dev.example.com does not exist, will create new listener rule
    ECS Service dev-dynamic-api-foo does not exist, will create new service
    Checking if record api-foo.dev.example.com. exists in zone Zxxxxxxxxx
    Successfully created CNAME: api-foo.dev.example.com -> dev-alb-api-dynamic-297517510.ap-northeast-1.elb.amazonaws.com
    Updating config file s3://example-com-config/endpoints.json, environment example-api-foo: nodeDomain -> api-foo.dev.example.com
    Existing environment not found, adding new
    Successfully updated config file
    Successfully finished creating environment dev-dynamic-api-foo

.. note::
    Create and delete commands update remote state DynamoDB Table. As with ``eden config push``, table will be created
    for you if it does not exist.

Check creation:

.. code-block:: console

    $ eden ls
    Profile api:
    dev-dynamic-api-foo api-foo.dev.example.com (last updated: 2019-11-20T19:44:10.179760)

.. note::
    This list is generated from remote state DynamoDB Table and not `environments JSON <eden_envs_json_>`_ file.
    Last updated timestamp is updated on creation and deploys as well.


Delete environment and check deletion:

.. code-block:: console

    $ eden delete -p api --name foo
    Updating config file s3://example-com-config/endpoints.json, delete environment example-api-foo: nodeDomain -> api-foo.dev.example.com
    Existing environment found, and the only optional key is nodeDomain,deleting environment
    Successfully updated config file
    Checking if record api-foo.dev.example.com. exists in zone Zxxxxxxxxx
    Found existing record api-foo.dev.example.com. in zone Zxxxxxxxxx
    Successfully removed CNAME record api-foo.dev.example.com
    ECS Service dev-dynamic-api-foo exists, will delete
    Successfully deleted service dev-dynamic-api-foo from cluster dev
    ELBv2 listener rule for target group arn:aws:elasticloadbalancing:ap-northeast-1:xxxxxxxxxx:targetgroup/dev-dynamic-api-foo/xxxxxxxxxxxx and host api-foo.dev.example.com found, will delete
    Deleted target group arn:aws:elasticloadbalancing:ap-northeast-1:xxxxxxxxxx:targetgroup/dev-dynamic-api-foo/xxxxxxxxxxxx
    Deleted all task definitions for family: dev-dynamic-api-foo, 1 tasks deleted total
    Successfully finished deleting environment dev-dynamic-api-foo

    $ eden ls
    No environments available

Moving to API
-------------

Both CLI and API manage their state in a DynamoDB Table. This table is only created by CLI. Furthermore, API can only
use "remote profiles", saved in state table. Before running API, make sure you pushed a profile to use with API by
running ``eden config --push``. If this table does not exist during API creation in Terraform, terraform apply will
fail.

API internals
^^^^^^^^^^^^^

eden API consists of:

1. Lambda function (the API itself)
2. API Gateway with API key for protecting API
3. DynamoDB Table for state management
   - Default table name is eden.

.. _`aws-eden-cli`: https://github.com/baikonur-oss/aws-eden-cli
.. _`lambda-eden-api`: https://github.com/baikonur-oss/terraform-aws-lambda-eden-api
.. _`aws-eden-core`: https://github.com/baikonur-oss/aws-eden-core


Creating eden API with Terraform
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: terraform

    module "eden_api" {
      source  = "baikonur-oss/lambda-eden-api/aws"
      version = "0.2.0"

      lambda_package_url = "https://github.com/baikonur-oss/terraform-aws-lambda-eden-api/releases/download/v0.2.0/lambda_package.zip"
      name                  = "eden"

       # eden API Gateway variables
      api_acm_certificate_arn     = "${data.aws_acm_certificate.wildcard.arn}"
      api_domain_name             = "${var.env}-eden.${data.aws_route53_zone.main.name}"
      api_zone_id                 = "${data.aws_route53_zone.main.zone_id}"

      endpoints_bucket_name = "somebucket"

      dynamic_zone_id       = "${data.aws_route53_zone.dynamic.zone_id}"
    }

.. warning::
   DynamoDB table for state management is created by aws-eden-cli.
   Make sure to run ``eden config --push`` with success at least once before terraform apply.

With multiple profiles, one eden API instance is enough for one account/region.
Refer to `profile section <eden_profiles_>`_ for more details.

eden API commands
^^^^^^^^^^^^^^^^^

eden has only two API commands: create and delete.

GET /api/v1/create
""""""""""""""""""

Required query parameters:
- name: environment name
- image_uri: ECR image URI to deploy, must be already pushed and must be in the same account (eden API will check for image availability before deploying)

Optional query parameters:
- profile: default value = "default". eden profile to use. Profiles include all settings necessary. Profiles can be created with `eden config --push` command. Refer to aws-eden-cli examples for more details.

GET /api/v1/delete
""""""""""""""""""

Required query parameters:
- name: environment name

Optional query parameters:
- profile: eden profile to use (default value = ``default``). Profiles include all settings necessary. Profiles can be created with ``eden config --push`` command (`see here for details <eden_profiles_>`_).


eden API Keys
^^^^^^^^^^^^^
eden API Terraform module creates one API Key for you.
You can check it from API Gateway console.

You will need to specify this key to access API.

Key must be provided as an HTTP header::

    x-api-key: YOURAPIKEY


API example
^^^^^^^^^^^

Let's run create API (with a remote profile called ``api``):

.. code-block:: console

    curl https://eden.example.com/api/v1/create?name=test-create&image_uri=xxxxxxxxxxxx.dkr.ecr.ap-northeast-1.amazonaws.com/servicename-api-dev:latest&profile=api -H "x-api-key:YOURAPIKEY"

Now let's look at logs that API Lambda Function has produced:

.. code-block:: text

    2019-04-08T20:32:05.151Z INFO     [main.py:check_cirn:382] Checking if image xxxxxxxxxxxx.dkr.ecr.ap-northeast-1.amazonaws.com/servicename-api-dev:latest exists
    2019-04-08T20:32:05.270Z INFO     [main.py:check_cirn:401] Image exists
    2019-04-08T20:32:05.446Z INFO     [main.py:create_env:509] Retrieved reference service arn:aws:ecs:ap-northeast-1:xxxxxxxxxxxx:service/dev/dev01-api
    2019-04-08T20:32:05.484Z INFO     [main.py:create_task_definition:58] Retrieved reference task definition from arn:aws:ecs:ap-northeast-1:xxxxxxxxxxxx:task-definition/dev01-api:15
    2019-04-08T20:32:05.557Z INFO     [main.py:create_task_definition:96] Registered new task definition: arn:aws:ecs:ap-northeast-1:xxxxxxxxxxxx:task-definition/dev-dynamic-test-create:1
    2019-04-08T20:32:05.584Z INFO     [main.py:create_target_group:112] Retrieved reference target group: arn:aws:elasticloadbalancing:ap-northeast-1:xxxxxxxxxxxx:targetgroup/dev01-api/9c68a5f91f34d9a4
    2019-04-08T20:32:05.611Z INFO     [main.py:create_target_group:125] Existing target group dev-dynamic-test-create not found, will create new
    2019-04-08T20:32:06.247Z INFO     [main.py:create_target_group:144] Created target group
    2019-04-08T20:32:06.310Z INFO     [main.py:create_alb_host_listener_rule:355] ELBv2 listener rule for target group arn:aws:elasticloadbalancing:ap-northeast-1:xxxxxxxxxxxx:targetgroup/dev-dynamic-test-create/b6918e6e5f10389d and host api-test.dev.example.com does not exist, will create new listener rule
    2019-04-08T20:32:06.361Z INFO     [main.py:create_env:554] ECS Service dev-dynamic-test-create does not exist, will create new service
    2019-04-08T20:32:07.672Z INFO     [main.py:check_record:414] Checking if record api-test.dev.example.com. exists in zone Zxxxxxxxxxxxx
    2019-04-08T20:32:08.133Z INFO     [main.py:create_cname_record:477] Successfully created ALIAS: api-test.dev.example.com -> dev-alb-api-dynamic-xxxxxxxxx.ap-northeast-1.elb.amazonaws.com
    2019-04-08T20:32:08.134Z INFO     [main.py:create_env:573] Successfully finished creating environment dev-dynamic-test-create

As state is managed in a remote DynamoDB table, you can check creation using eden CLI:

.. code-block:: console

    $ eden ls
    Profile api:
    dev-dynamic-test-create api-test.dev.example.com (last updated: 2019-04-08T20:32:08.134469)

Now let's delete this environment by running:

.. code-block:: console

    curl https://eden.example.com/api/v1/delete?name=test&profile=api -H "x-api-key:YOURAPIKEY"

API Lambda logs will look like this:

.. code-block:: text

    2019-04-10T23:11:38.515Z INFO     [main.py:check_record:495] Checking if record api-test.dev.example.com. exists in zone Zxxxxxxxxxxxx
    2019-04-10T23:11:38.752Z INFO     [main.py:check_record:506] Found existing record api-test.dev.example.com. in zone Zxxxxxxxxxxxx
    2019-04-10T23:11:38.996Z INFO     [main.py:delete_cname_record:596] Successfully removed ALIAS record api-test.dev.example.com
    2019-04-10T23:11:39.245Z INFO     [main.py:delete_env:665] ECS Service dev-dynamic-test exists, will delete
    2019-04-10T23:11:39.401Z INFO     [main.py:delete_env:670] Successfully deleted service dev-dynamic-test from cluster dev
    2019-04-10T23:11:39.573Z INFO     [main.py:delete_alb_host_listener_rule:397] ELBv2 listener rule for target group arn:aws:elasticloadbalancing:ap-northeast-1:xxxxxxxxxxxx:targetgroup/dev-dynamic-test/xxxxxxxx and host api-test.dev.example.com found, will delete
    2019-04-10T23:11:40.483Z INFO     [main.py:delete_env:697] Deleted all task definitions for family: dev-dynamic-test, 5 tasks deleted total
    2019-04-10T23:11:40.483Z INFO     [main.py:delete_env:700] Successfully finished deleting environment dev-dynamic-test
