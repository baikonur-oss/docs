.. _repository_index:

Baikonur Project repository index
=================================

.. _repository_index_logging:

:ref:`baikonur_logging_top` Terraform Modules
---------------------------------------------

lambda-kinesis-to-fluent_
^^^^^^^^^^^^^^^^^^^^^^^^^

Transfer data from Kinesis Data Streams to Fluent endpoint

lambda-kinesis-to-s3_
^^^^^^^^^^^^^^^^^^^^^

Save data from Kinesis Data Streams to S3

lambda-kinesis-to-es_
^^^^^^^^^^^^^^^^^^^^^

Add data from Kinesis Data Streams to Elasticsearch

lambda-es-cleaner_
^^^^^^^^^^^^^^^^^^

Automatically clean old indices in Elasticsearch Service

lambda-kinesis-forward_
^^^^^^^^^^^^^^^^^^^^^^^

Kinesis Data Streams to Kinesis Data Streams forwarder/router module

.. _`lambda-kinesis-to-fluent`: https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-to-fluent
.. _`lambda-kinesis-to-s3`: https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-to-s3
.. _`lambda-kinesis-to-es`: https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-to-es
.. _`lambda-es-cleaner`: https://github.com/baikonur-oss/terraform-aws-lambda-es-cleaner
.. _`lambda-kinesis-forward`: https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-forward

.. _repository_index_eden:

:ref:`eden_top`
---------------

aws-eden-cli_
^^^^^^^^^^^^^
eden CLI

lambda-eden-api_
^^^^^^^^^^^^^^^^
eden API

Tools and libraries
-------------------

amazon_kinesis_utils_
^^^^^^^^^^^^^^^^^^^^^^
A Python library of useful utilities for Amazon Kinesis. Extensively used by `logging modules <repository_index_logging_>`_.

aws-eden-core_
^^^^^^^^^^^^^^
Internal library for `eden modules <repository_index_eden_>`_.

Other Terraform Modules for AWS
-------------------------------

`iam-nofile`_
^^^^^^^^^^^^^

Module to make it easier to create AWS IAM Roles. You can write role policy document with
inline(heredoc) syntax, so you do not have to use `template rendering`_ to use variables.

.. raw:: html

    <script src="https://gist.github.com/prog893/bd5c7c118ea24e98e3240e20f68f9bb4.js"></script>

`fargate-scheduled-task`_
^^^^^^^^^^^^^^^^^^^^^^^^^

Create ECS scheduled tasks with CloudWatch Events easily (for batch apps, or anything else that runs on routine)

.. raw:: html

    <script src="https://gist.github.com/prog893/5e70f1a1e79035343e0435bbd207b770.js"></script>


.. _`template rendering`: https://www.terraform.io/docs/providers/template/d/file.html
.. _`fargate-scheduled-task`: https://github.com/baikonur-oss/terraform-aws-fargate-scheduled-task
.. _`iam-nofile`: https://github.com/baikonur-oss/terraform-aws-iam-nofile
.. _`amazon_kinesis_utils`: https://github.com/baikonur-oss/amazon-kinesis-utils
.. _`aws-eden-cli`: https://github.com/baikonur-oss/aws-eden-cli
.. _`lambda-eden-api`: https://github.com/baikonur-oss/terraform-aws-lambda-eden-api
.. _`aws-eden-core`: https://github.com/baikonur-oss/aws-eden-core
