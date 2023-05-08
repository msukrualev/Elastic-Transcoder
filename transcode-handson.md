# Transcoding Video with S3 and Elastic Transcoder

## Introduction

In this hands-on AWS lab, we will convert video files to different formats using Lambda, S3, and Amazon Elastic Transcoder.

Elastic Transcoder is a media service designed to be a highly scalable, easy-to-use, and cost-effective way to convert (or "transcode") media files from their source format into versions that will play back on devices like smartphones, tablets, and PCs.

In this lab, we will upload 4K Ultra HD sample videos to an S3 bucket and configure an Elastic Transcoder pipeline to automatically convert them to different formats. We'll set all this up with our own custom Lambda function — written in Python — using the Boto3 SDK.


## Log in to the AWS Management Console
Make sure you are using the us-east-1 (N. Virginia) region.

Create S3 buckets
- Create 3 buckets with default settings. Names should look similar to the following:
mybucket-source
mybucket-transcoded
mybucket-thumbnails


Subscribe to an SNS Topic
- In the AWS Management Console, navigate to the SNS service.
- Click Topics in the left sidebar.
- Select Standard Type.
- Create the Transcoder topic.
- Create Subscription to topic.
- In the Protocol dropdown box, select Email.
- In the Endpoint box, type an email address you can use to receive the notification.
- Click Create subscription.
- Go to your email application and open the message from AWS Notifications.
- Click the link to confirm your subscription.
- Go back to your SNS browser tab, and open the Transcoder topic to verify that the subscription is confirmed.

Create an Elastic Transcoder Pipeline
- In the AWS Management Console, navigate to the Elastic Transcoder service.
- Click Create a new Pipeline.
Under Create New Pipeline, configure the following settings:
- Pipeline Name: MyPipeline (or any name you like)
- Input Bucket: (Select the bucket with source in its name.)
- IAM Role: Create console default role
Under Configuration for Amazon S3 Bucket for Transcoded Files and Playlists, configure the following settings:
- Bucket: (Select the bucket with transcoded in its name.)
- Storage Class: Standard
Under Configuration for Amazon S3 Bucket for Thumbnails, configure the following settings:
- Bucket: (Select the bucket with thumbnails in its name.)
- Storage Class: Standard
Expand the Notifications menu.
- Select Use an existing SNS topic for all of the events.
- For Select a Topic, choose Transcoder from the dropdown for all of the events.
- Click Create Pipeline.
Note the Pipeline ID. (This is available in the pipeline details and is a different value than the pipeline name.)

Create a Lambda Function
- In the AWS Management Console, navigate to the Lambda service.
- Click Create a function.
Choose the Author from scratch option, and configure the following settings:
- Name: TranscodeVideo (or any name you like)
- Runtime: Python 3.7
- Role: Create a custom role
- Go to IAM service and first create a policy. Paste the policy json file code into place.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Action": [
        "elastictranscoder:*",
        "s3:*"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```


Name: Lambda-s3-transcode-log-policy
Create policy.

- Create role using the policy.
Name: Lambda-s3-transcode-log-role
Create role.
- Go back to your Lambda Management Console browser tab, select the role and click Create function.
- On the fucntions main page click to add a trigger event, and select S3.
Scroll down the page to the Configure triggers section, and configure the following settings:
- Bucket: (Select the bucket with source in its name.)
- Event type: All object create events
Click Add to add a trigger to the function.

- Scroll down the page to the function editor, and remove the default code.
- Paste the contents of lambda-function.py source code into the code editor. Then Deploy.

```python
from datetime import datetime
import json
import urllib.parse
import os
import boto3

PIPELINE_ID = os.environ['PIPELINE_ID']

transcoder = boto3.client('elastictranscoder')
s3 = boto3.resource('s3')


def lambda_handler(event, context):
    print("Received event: " + json.dumps(event))

    # Get the object from the event
    key = urllib.parse.unquote_plus(
        event['Records'][0]['s3']['object']['key'], encoding='utf-8')

    filename = os.path.splitext(key)[0]  # filename w/o extension

    # Create a job
    job = transcoder.create_job(
        PipelineId=PIPELINE_ID,
        Input={
            'Key': key
        },
        Outputs=[
            {
                'Key': filename + '-1080p.mp4',
                'ThumbnailPattern': filename + '-{resolution}-{count}',
                'PresetId': '1351620000001-000001'  # Generic 1080p
            },
            {
                'Key': filename + '-720p.mp4',
                'ThumbnailPattern': filename + '-{resolution}-{count}',
                'PresetId': '1351620000001-000010'  # Generic 720p
            }
        ]
    )

    print("start time={}".format(datetime.now().strftime("%H:%M:%S.%f")[:-3]))
    print("job={}".format(job))
    job_id = job['Job']['Id']

    # Wait for the job to complete
    waiter = transcoder.get_waiter('job_complete')
    waiter.wait(Id=job_id)
    end_time = datetime.now().strftime("%H:%M:%S.%f")[:-3]
    print("end time={}".format(end_time))
```

- Switch back to your Elastic Transcoder tab, and click the magnifying glass icon to view the details for our pipeline.
- Copy the Pipeline ID to your clipboard, and switch back to your Lambda Management Console browser tab.
- Click Configuration and then scroll down to the Environment variables section,click Edit and set the following variable:
Key: PIPELINE_ID | Value: (Paste the pipeline ID from the pipeline summary page)
Click Save.

>>
Upload Video for Transcoding
- In the AWS Management Console, navigate to the S3 service.
- Open the S3 bucket with source in its name.
- Click Upload.
- Click Add files.
- Select one of the 4K sample video files you downloaded to your local machine.
- Click Upload.

- Open your email application, and check for an email notification from Elastic Transcoder.
- Go back to the S3 Management Console, and open the S3 bucket with transcoded in its name.
- Verify that the 1080p and 720p versions of your uploaded video are present.
- Select one of the transcoded versions of the uploaded video, and verify that it is in the correct format.
- Go back to the S3 home page, and select the S3 bucket with thumbnails in its name.
- Select the .png image that was created, and click Open to verify that the thumbnail was successfully created.

Conclusion
Congratulations, you've successfully completed this lab!