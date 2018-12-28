SES Mailer
==========
Python-based (Boto) mailer for AWS Simple Email Service (SES).  Demonstrates how to send email faster using AWS SES without a backend.

Based on the excellent code from AWS at https://github.com/awslabs/aws-support-tools/tree/119b4a6c22472515008a049506d6dbfcdee59ac8/SES/SESMailer, with the following features added:

* Recipient email address validation
* An attempt at idempotence, so that multiple successive runs of the script do not result in sending duplicate messages to recipients
    * Suitable for invocation by AWS Lambda
* Displays statistics at script conclusion
    * Number of messages sent
    * Number of message resends skipped (due to idempotence)
* Improved error/exception handling and reporting
    * Print all errors, instead of uploading to S3
* Improved cleanup of inputs from S3 bucket, to prevent accidental future sending mistakes
* Simplified execution flow when using a single thread
* Various bugfixes


How it works
------------
SES Mailer uses AWS Lambda to send mass email. It is invoked by S3 PUT event when a mailing list is dropped into the designated bucket. The function takes its message content from a HTML and plain text message file dropped in the same bucket. It sends multipart/alternative formatted emails.

Usage
-----
1. Deploy `ses_mailer.py` on AWS Lambda with a timeout setting of 5 minutes. Make sure the lambda Role has
   S3 read/write permissions to the bucket and `ses:SendRawEmail` permission. Configure Lambda environment variables:
      * `REGION=us-east-1`
      * `MAX_THREADS=1`
      * `TEXT_MESSAGE_FILE=email_body.txt`
      * `HTML_MESSAGE_FILE=email_body.html`
2. Create a S3 bucket and set `put` event to trigger this lambda function.
3. In the S3 events configuration, set the event suffix to `.gz`.
4. Write your HTML-formatted email in file **email_body.html** and upload to S3 bucket.
5. Write the plain-text version of your email message in file **email_body.txt** and upload to S3 bucket.
6. Create a mailing list file (e.g. **mailing_list_14032016.csv**) with contents in the below csv format. **Don't** include first row as header fields.

    ```
    Sender Name <me@example.com>, Recipient Name <you@example.com>, subject
    ```

7. Compress the file using gzip. e.g. `gzip -kf mailing_list_14032016.csv` creates `mailing_list_14032016.csv.gz`
8. Upload the gzipped file **mailing_list_14032016.csv.gz** to the S3 bucket and it will trigger this lambda function.
9. This function will start sending email to all addresses in the csv file and log failures to the console

**Tip:** You can send even faster by splitting email list into multiple smaller csv files when the number of addresses exceed over a few 100,000s or increase `MAX_THREADS` environment variable value to something higher depending on your SES TPS limit.
