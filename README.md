# awsbill2graphite

Converts AWS hourly billing CSVs to Graphite metrics

## Prep

First of all, you'll need to have hourly billing reports enabled. You can do this
through the AWS billing control panel.

In order to prevent Graphite from creating giant, mostly-zero data files, set the
following in `storage-schemas.conf`:

    [awsbill]
    priority = 256
    pattern = ^awsbill\.
    retentions = 1h:3650d

## Usage

First set the following environment variables:

* `AWSBILL_REPORT_PATH`: The path where the report lives. If downloading from S3, this
  should be `s3://` followed by the bucket name followed by the "Report path" as defined
  in the AWS billing control panel. If reading a local file, it should start with
 `file://` and give the path to an hourly billing CSV file.
* `AWS_ACCESS_KEY_ID`: The identifier for an AWS credentials pair that will enable access
  to the bucket with billing reports in it. If you're using a local file instead of
  downloading the report from S3, you can omit this.
* `AWS_SECRET_ACCESS_KEY`: The secret access key that corresponds to `AWS_ACCESS_KEY_ID`.
  If you're using a local file instead of downloading the report from S3, you can omit
  this.
* `AWSBILL_GRAPHITE_URL`: The URL of the Graphite server to which to write metrics. If
  instead you want to output metrics to stdout, set this environment variable to `stdout`.
* `AWSBILL_METRIC_PREFIX`: The prefix to use for metrics written to Graphite. If absent,
  metrics will begin with "`awsbill.`". If you set this, you should modify the `[awsbill]`
  stanza you added to Graphite's `storage-schemas.conf` accordingly.
* `AWSBILL_TAGS`: The (comma-separated) tags to produce metrics for. If you don't want
  to produce metrics for any tags, leave this environment variable empty.

Then run

    awsbill2graphite.py

This will produce metrics named like so:

    PREFIX.REGION.ec2-instance.t2-micro
    PREFIX.REGION.ec2-instance.c4-2xlarge
    PREFIX.REGION.ec2-other.snapshot-storage
    PREFIX.REGION.ec2-other.piops
    PREFIX.REGION.rds.m4-medium

If tags are specified in `AWSBILL_TAGS`, then additional metrics will be populated, named
like so:

    PREFIX.REGION.TAG_NAME.TAG_VALUE.ec2-instance.t2-micro
    PREFIX.REGION.TAG_NAME.TAG_VALUE.ec2-instance.c4-2xlarge
    PREFIX.REGION.TAG_NAME.TAG_VALUE.ec2-other.snapshot-storage
    PREFIX.REGION.TAG_NAME.TAG_VALUE.ec2-other.piops
    PREFIX.REGION.TAG_NAME.TAG_VALUE.rds.m4-medium

Each metric will have a data point every hour. This data point represents the total amount
charged to your account for the hour _previous_ to the data point's timestamp.
