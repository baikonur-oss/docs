.. _baikonur_logging_top:

Baikonur Logging with Amazon Kinesis and AWS Lambda
===================================================

Prerequisites
-------------

Why Kinesis?
^^^^^^^^^^^^

Kinesis Data Streams is a high-throughput, low latency, fully managed service for working with streaming data.
A single shard provides `1 MB/s read and 2 MB/s write capacity <kinesis_quotas_>`_.
Data put to a Kinesis Data Stream is saved for a period specified in the data retention period parameter.
Thus, for capacity planning, it is enough to know peak data throughput.

.. note::
    Kinesis Data Streams do not have auto-scaling capabilities.
    You can have your Kinesis Data Stream shards scale automatically by using `amazon-kinesis-scaling-utils`_.

Why Kinesis with Lambda?
^^^^^^^^^^^^^^^^^^^^^^^^

Using Lambda functions to process data on Kinesis Streams (with event source mapping)
reduces the amount of code and resources you have to manage:

1. With Lambda, you do not have to manage servers, OS, etc.
2. No need to write the logic for reading data from Kinesis Data Stream.
   Data batches are passed to Lambda functions on execution via ``event`` object.

3. Event source mapping manages processed data position on Kinesis Data Stream for you.
   It keeps track of until what position on each shard data is already successfully processed.
   The position is only updated if Lambda finishes without errors.

3. If a Lambda function mapped to a Kinesis Data Stream finishes with an error, it is executed again with the same batch
   until execution succeeds or data expires.
   Records are removed from Kinesis Data Stream only when the data retention period for those records expires.

   - Thus, you do not have to write retry logic in Lambda.
     Although, it may be a good idea to save data that is failing after n retries to a queue (or S3) and return without
     errors so that the data processing pipeline does not stop.

Why use Kinesis and Lambda for logging?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Kinesis and Lambda combination can replace self-managed log clusters with managed services while retaining control of
log procession.

By using Kinesis with Lambda, we can create a modular, extendable, scalable logging architecture.
Log transfer reliability may improve as well: data written to a Kinesis stream successfully will not get lost.

Common Schema requirements
--------------------------

“Baikonur Logging architecture" is any architecture using Kinesis Data Streams in conjunction with one or more of the
following Baikonur Logging Lambda modules:

- terraform-aws-lambda-kinesis-forward_
- terraform-aws-lambda-kinesis-to-es_
- terraform-aws-lambda-kinesis-to-s3_
- terraform-aws-lambda-kinesis-to-fluent_

These modules have the following Common Schema requirements:

- All data must be JSON. The root element type must be an object.
- All data must include the following keys:

    - Data type identifier. (default key name: ``log_type``)
    - Any unique identifier, e.g. ``uuid.uuid4()`` (default key name: ``log_id``)
    - Any timestamp supported by dateutil. (default key name: ``time``)

.. note::
    All key names are customizable.

Common Schema requirements allow us for:

1. Easier parsing
2. Better interoperability between different Lambda modules.

    - You can attach different modules to the same Kinesis Data Stream.

3. Ability to create behavior based on keys that are part of Common Schema requirements:

    - One of the most important features is the ability to filter logs based on data type field value.
    - terraform-aws-lambda-kinesis-to-s3_ requires log_id to ensure filename uniqueness and time key to separate logs
      by date.
    - terraform-aws-lambda-kinesis-to-es_ requires time key to create daily indices in Elasticsearch
      (e.g. index-20200314).

.. note::
    As long as data meets Common Schema requirements, architectures and modules described in this documentation are
    applicable to any data transfer that needs to be fast and reliable, for example, inter-microservice communication.

Architecture examples
---------------------

One stream - one destination
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create a Kinesis Data Stream for each destination.
For example, if we want to save all log data to S3 and only some to Elasticsearch, create two streams,
deploy terraform-aws-lambda-kinesis-to-s3_ and terraform-aws-lambda-kinesis-to-es_ modules and map them to the respective
streams::

       Kinesis API
           v
    [App] ---> [KDS "s3"] ---> [kinesis-to-s3] ---> S3
         \
          ---> [KDS "es"] ---> [kinesis-to-es] ---> ES


If you want to save logs to both S3 and Elasticsearch, write data to both streams.

One stream - multiple destinations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the example above, we have to write logs we want to save to Elasticsearch to both streams.
We can further improve this by adding terraform-aws-lambda-kinesis-to-s3_ to stream for Elasticsearch as well::

       Kinesis API
           v
    [App] ---> [KDS "s3"] ---> [kinesis-to-s3] ---> S3
         \
          ---> [KDS "es"] ---> [kinesis-to-es] ---> ES
                         \
                          ---> [kinesis-to-s3] ---> S3

Now we write each log event at most once.

.. _kinesis_routing_pattern:

Kinesis routing pattern
^^^^^^^^^^^^^^^^^^^^^^^

Write data to a single Kinesis stream (a “router”).
Create multiple output streams, each for a destination.
We can use forwarder modules (terraform-aws-lambda-kinesis-forward_) with whitelists to create an architecture similar
to the `Publish-subscribe pattern <https://en.wikipedia.org/wiki/Publish–subscribe_pattern>`_, where a topic is a value
in the type field, and each output stream represents a subscription group::

       Kinesis API
           v
    [App] ---> [KDS "router"] ---> [kinesis-forward] ---> [KDS "A"]
                             \
                              ---> [kinesis-forward] ---> [KDS "B"]
                              \
                               --> [kinesis-forward] ---> [KDS "C"]

This pattern may also be useful for inter-microservice communication.

Each of output streams may have their own Lambda modules or subscribers::

       Kinesis API
           v
    [App] ---> [KDS "router"] ---> [kinesis-forward] ---> [KDS "A"] ---> [S3]
                             \
                              ---> [kinesis-forward] ---> [KDS "B"] ---> [ES]
                              \
                               --> [kinesis-forward] ---> [KDS "C"] <--- [External subscriber]



Kinesis routing pattern with CloudWatch Logs subscription filters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In addition to `Kinesis Routing pattern <kinesis_routing_pattern_>`_, use CloudWatch Logs subscription filters to write
data to the “router” stream.
Doing so will free you from having to write PutRecord/PutRecords logic in your application if you already output logs to
CloudWatch.
For instance, if you are using awslogs logging driver in ECS, using subscription filter may look like this::

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

.. _`kinesis_quotas`: https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html
