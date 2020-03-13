.. _baikonur_logging_top:

Baikonur Logging with Amazon Kinesis and AWS Lambda
===================================================

Prerequisite
------------

Why Kinesis?
^^^^^^^^^^^^

Kinesis Data Streams is a high-throughput, low latency, fully managed service for working with streaming data.

A single shard provides `1 MB/s read and 2 MB/s write capacity <kinesis_quotas_>`_. Data added to a Kinesis Data Stream
is kept on stream for a period of time specified in data retention period parameter. Thus, for capacity planning it is
enough to know peak data throughput.

.. note::
    Kinesis Data Streams do not have auto-scaling capabilities. With `amazon-kinesis-scaling-utils`_ you can have your
    Kinesis Data Stream shards scale automatically.

Why Kinesis with Lambda?
^^^^^^^^^^^^^^^^^^^^^^^^

By using Lambda for processing data on Kinesis streams (with event source mapping), several good things happen:

1. With Lambda, you do not have to manage servers etc.
2. Lambda backend does position management for you: it keeps track of until what position on every shard data is already
   processed, and only updates position if Lambda finishes without errors.

   - Moreover, you do not have to perform SubscribeToShard etc., every Lambda is executed with its' data batch to
     process in ``event`` object.

3. If Lambda finishes with error, Lambda is restarted with the same batch until it succeeds or data is deleted (data
   retention period passes since data was added to stream).
    - No need to write retry logic in Lambda. Though, it may be a good idea to data that is failing after ``n`` retries
      and move on, so that data processing does not stop.

Why use Kinesis and Lambda for logging?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Kinesis and Lambda combo can be used as a solution to replace self-managed log clusters with a managed services,
while still have control of how data is processed. By using Kinesis and Lambda it is possible to create a modular,
extendable, scalable architecture. Log data transfer reliability may be improved as well: data written to a Kinesis
stream successfully does not get lost.

Common Schema requirements
--------------------------

"Baikonur Logging with Amazon Kinesis and AWS Lambda" can be defined as any architecture using Kinesis Data Streams
in conjunction with one or more of the following Baikonur Lambda Modules:

- terraform-aws-lambda-kinesis-forward_
- terraform-aws-lambda-kinesis-to-es_
- terraform-aws-lambda-kinesis-to-s3_
- terraform-aws-lambda-kinesis-to-fluent_

These modules have the following common schema requirements:

- All data must be JSON, root type must be object
- All data must include the following keys:

    - ``log_type``: Log type identifier
    - ``log_id``: Any unique identifier (e.g. ``uuid.uuid4()``)
    - ``time``: Any timestamp supported by dateutil.parser.parse_

.. note::
    Key names are customizable for each Lambda module

Common schema requirements are derived from following needs:

1. Easier parsing
2. Interoperability between different Lambda modules

    - Different modules can be attached to a single Kinesis Data Stream
      and work on same data as long as data are JSON-objects and common schema requirements are met.

3. Ability to create behaviour based on keys in common schema

    - One of the most important features is ability to apply whitelist on ``log_type`` field to, for instance,
      ignore logs other than those specified.

``log_id`` and ``time`` keys are required by terraform-aws-lambda-kinesis-to-s3_ to ensure unique filenames,
and terraform-aws-lambda-kinesis-to-es_ for creating daily index names in Elasticsearch (e.g. index-name-20200314).
Additionally, these fields are useful when troubleshooting (e.g. searching for specific log event,
digging into log data by time).

.. note::
    As long as common schema requirements are met, this design pattern and modules mentioned here can be applied to any
    data transfer that needs to be fast and reliable, for example inter-microservice communication.

Architecture examples
---------------------

Single destination from single stream
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create a Kinesis Data Stream for each destination. For example, if we want to save all logs to S3 and only some of them
to Elasticsearch, create two streams: for S3 and Elasticsearch. Add terraform-aws-lambda-kinesis-to-s3_ and
terraform-aws-lambda-kinesis-to-es_ for the respective streams. Put logs to each::

       Kinesis API
           v
    [App] ---> [KDS "s3"] ---> [kinesis-to-s3] ---> S3
         \
          ---> [KDS "es"] ---> [kinesis-to-es] ---> ES

Multiple destinations from single stream
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In example above, we have to write logs we want to save to Elasticsearch to both streams. We can further improve this
by adding terraform-aws-lambda-kinesis-to-s3_ to stream for Elasticsearch as well::

       Kinesis API
           v
    [App] ---> [KDS "s3"] ---> [kinesis-to-s3] ---> S3
         \
          ---> [KDS "es"] ---> [kinesis-to-es] ---> ES
                         \
                          ---> [kinesis-to-s3] ---> S3

Now we only write each log event once to either stream.

.. _kinesis_routing_pattern:

Kinesis routing pattern
^^^^^^^^^^^^^^^^^^^^^^^

Write data to a single Kinesis stream (a "router"). Create multiple output streams, each for a destination.
Forwarders (terraform-aws-lambda-kinesis-forward_) with whitelists can be used to create a
`Publish-subscribe pattern`_-like architecture (topic is specified in type field, and each output stream is a
subscription group)::

       Kinesis API
           v
    [App] ---> [KDS "router"] ---> [kinesis-forward] ---> [KDS "A"]
                             \
                              ---> [kinesis-forward] ---> [KDS "B"]
                              \
                               --> [kinesis-forward] ---> [KDS "C"]

This pattern may also be useful for inter-microservice communication.

Each of output streams may have their own Lambda modules or subscribers. For example::

       Kinesis API
           v
    [App] ---> [KDS "router"] ---> [kinesis-forward] ---> [KDS "A"] ---> [S3]
                             \
                              ---> [kinesis-forward] ---> [KDS "B"] ---> [ES]
                              \
                               --> [kinesis-forward] ---> [KDS "C"] <--- [External subscriber]



Kinesis routing pattern with CloudWatch Logs subscription filters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In addition to kinesis_routing_pattern_, use CloudWatch Logs subscription filters to input data to "router" stream.
Doing so will free you from having to write PutRecord/PutRecords logic in your application if you already output logs to
CloudWatch. For instance, if you are using ``awslogs`` logging driver in ECS, using subscription filter will look like::

     stdout->awslogs      Subscription filter
           v                      v
    [App] ---> [CloudWatch Logs] ---> [KDS "router"] ---> [kinesis-forward] ---> [KDS "A"] ---> [S3]
                                                    \
                                                     ---> [kinesis-forward] ---> [KDS "B"] ---> [ES]
                                                     \
                                                      --> [kinesis-forward] ---> [KDS "C"] <--- [External subscriber]


.. _dateutil.parser.parse: https://dateutil.readthedocs.io/en/stable/parser.html#dateutil.parser.parse
.. _terraform-aws-lambda-kinesis-forward: https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-forward
.. _terraform-aws-lambda-kinesis-to-es: https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-to-es
.. _terraform-aws-lambda-kinesis-to-s3: https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-to-s3
.. _terraform-aws-lambda-kinesis-to-fluent: https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-to-fluent
.. _amazon-kinesis-scaling-utils: https://github.com/awslabs/amazon-kinesis-scaling-utils

.. _`Publish-subscribe pattern`: https://en.wikipedia.org/wiki/Publish–subscribe_pattern

.. _`kinesis_quotas`: https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html
