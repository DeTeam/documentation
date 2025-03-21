---
id: export
title: Export - Temporal Cloud feature guide
sidebar_label: Export
sidebar_position: 9
description: Workflow History export allows users to export Closed Workflow Histories to a user's Cloud Storage Sink.
slug: /cloud/export
toc_max_heading_level: 4
keywords:
- explanation
- how-to
- operations
- temporal cloud
- term
- verify
- workflow history export
tags:
- explanation
- how-to
- operations
- temporal-cloud
- term
- verify
- workflow-history-export
---


:::tip Support, stability, and dependency info
- Workflow History Export is in a Public Preview release status for Temporal Cloud.

:::

The Workflow History Export feature allows users to export Closed Workflow Histories to a user's Cloud Storage Sink.

The Export feature in Temporal Cloud provides the following benefits:

- Preserve complete set of Workflow History data in [proto format](https://github.com/temporalio/api/blob/master/temporal/api/export/v1/message.proto) for compliance and auditing purposes.
- Enables feeding Workflow History into data warehouses for analytics.

For pricing information, see [Temporal Cloud Pricing](/cloud/pricing).

## Supported integrations for Workflow History Export {#supported-integrations}

The Workflow History Export feature allows users to export Closed Workflow Histories to [Amazon Simple Storage Service (S3)](https://docs.aws.amazon.com/s3/) as the destination.

You can enable Workflow History Export per Namespace in Temporal Cloud.

### Prerequisites {#prerequisites}

Before configuring the Export Sink, ensure you have the following:

1. An AWS Account with write permission to an S3 bucket.
2. An AWS S3 bucket.
   1. The S3 bucket must reside in the same region as your Namespace.
3. (optional) A KMS ARN associated with the S3 bucket.

## Configure Workflow History Export {#configure}

**How to configure Workflow History Export**

You can use either the [Temporal Cloud UI](#using-temporal-cloud-ui) or [tcld](#using-tcld) to configure the Workflow History Export.

### Using Temporal Cloud UI

The following steps guide you through setting up Workflow History Export using the Temporal Cloud UI.

![](/img/export-sink-ui.png)

The Temporal Cloud UI provides two ways for configuring Workflow History Export:

- [Automated setup](#automated-setup) (recommended): The Cloud UI launches the AWS CloudFormation Console to create a stack, with write permission to the S3 bucket.
- [Manual setup](#manual-setup): The Cloud UI provides a CloudFormation template for users to manually configure a CloudFormation stack.

#### Automated setup

The automated setup creates a CloudFormation stack with write permission to the S3 bucket.
[Verify the export setup](#verify) before saving the configuration.

1. Open the Temporal Cloud UI and navigate to the Namespace you want to configure.
2. Select **Configure** from the **Export** card.
3. Provide the following information to configure the export sink and then select **Create and launch stack**:
   1. Name: A name for the export sink.
   2. AWS S3 Bucket Name: The name of the configured AWS S3 bucket to send Closed Workflow Histories to.
   3. AWS Account ID: The AWS account ID.
   4. Role Name: The name of the AWS IAM role to use for the CloudFormation stack that has write permission to the S3 bucket.
   5. KMS ARN: (optional) The ARN of the AWS KMS key to use for encryption of the exported Workflow History.
4. You will be taken to the CloudFormation Console to create the stack with pre-populated information.
   1. Review the information and then select **Create stack**.

#### Manual setup

The manual setup provides a CloudFormation template to manually configure a CloudFormation stack.

1. Open the Temporal Cloud UI and navigate to the Namespace you want to configure.
2. Select **Configure** from the **Export** card.
3. Select **Manual** from **Access method**.
   1. Enter the Template URL into your web browser to download your copy of the CloudFormation template.
   2. Configure the CloudFormation template for your export sink.
   3. Follow the steps in the [AWS documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-console-create-stack-template.html) by uploading the template to the CloudFormation console.

### Using tcld

Run the `tcld namespace export s3 create` command and provide the following information.

- `--namespace` : The Namespace to configure export for.
- `--sink-name`: The name of the export sink.
- `--role-arn`: The ARN of the AWS IAM role to use for the CloudFormation stack that has write permission to the S3 bucket.
- `--s3-bucket-name`: The name of the AWS S3 bucket.

For example:

```command
tcld namespace export s3 create --namespace "your-namespace.your-account" --sink-name "your-sink-name" --role-arn "arn:aws:iam::123456789012:role/test-sink" --s3-bucket-name "your-aws-s3-bucket-name”
```

Retrieve the status of this command by running the `tcld namespace export s3 get` command.

For example:

```command
tcld namespace export s3 get --namespace "your-namespace.your-account" --sink-name "your-sink-name"
```

The following is an example of the output:

```json
{
  "name": "your-sink-name",
  "resourceVersion": "a6442895-1c07-4da4-aaca-58d57d338345",
  "state": "Active",
  "spec": {
    "name": "your-sink-name",
    "enabled": true,
    "destinationType": "S3",
    "s3Sink": {
      "roleName": "your-export-test",
      "bucketName": "your-export-test",
      "region": "us-east-1",
      "kmsArn": "",
      "awsAccountId": "123456789012"
    }
  },
  "health": "Ok",
  "errorMessage": "",
  "latestDataExportTime": "0001-01-01T00:00:00Z",
  "lastHealthCheckTime": "2023-08-14T21:30:02Z"
}
```

## Verify export setup {#verify}

**How to verify that the Export feature is set up correctly**

From the Export configuration page, select **Verify**.
This action checks if Temporal can successfully write a test file to the sink.

If everything is configured correctly, you will see a `Success` status indicating Temporal has written to your sink.

## Monitor export progress {#monitor}

**How to monitor the History export progress**

Once you've finalized the setup, here's how to monitor the export progress:

1. **Export Job Execution**:
   - **Schedule**: The Export job is scheduled to run on an hourly basis, starting at 10 minutes past each hour.
   - **Duration**: The time taken for the export process can vary, so it might not be instantaneous.

2. **Checking the S3 Bucket**:
   - **Files Arrival**: Post the initial hour of setting up, inspect your S3 bucket.
     You should see the exported Workflow History files.
     These files encapsulate the relevant data from the Closed Workflows.
   - **Directory Structure**: Your exported files will adhere to the following naming convention and path:

   ```command
   s3://[bucket-name]/temporal-workflow-history/export/[Namespace]/[Year]/[Month]/[Day]/[Hour]/[Minute]/
   ```

3. **Delivery guarantee**:
   - At least once delivery.
   - Each Closed Workflow is expected to be found in the exported file within one to four hours.

4. **UI insights**:
   - **Last Successful Export**: This displays the timestamp of the most recent successful export.
     It's an essential indicator of the last time your export process completed without any hitches.
   - **Last Status Check**: This reflects the timestamp of the latest internal Workflow check.
     This internal check routinely evaluates the health and status of the Export mechanism, ensuring its uninterrupted functioning.

5. **Usage monitoring**:
   - Actions from the Export Job are included in the Usage UI.
   - **Metrics**: Export related metrics are available in `temporal_cloud_v0_total_action_count` with the label `is_background=“true”`. For more information, see [Cloud metrics](/cloud/metrics/).

For optimal results, review the S3 bucket for any new exported files and refer to the UI insights.
This dual check ensures you remain abreast of the export progress and any potential issues.

<!--  starting on `2024/03/01` UTC actions are charged. -->

