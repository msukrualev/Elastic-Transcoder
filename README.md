# Transcoding Video with S3 and Elastic Transcoder

## Introduction

In this hands-on AWS lab, we will convert video files to different formats using Lambda, S3, and Amazon Elastic Transcoder.

Elastic Transcoder is a media service designed to be a highly scalable, easy-to-use, and cost-effective way to convert (or "transcode") media files from their source format into versions that will play back on devices like smartphones, tablets, and PCs.

In this lab, we will upload 4K Ultra HD sample videos to an S3 bucket and configure an Elastic Transcoder pipeline to automatically convert them to different formats. We'll set all this up with our own custom Lambda function — written in Python — using the Boto3 SDK.