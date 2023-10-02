# Scanning S3 for Personally Identifiable Information (PII) using Macie

## Overview

This GitHub project aims to demonstrate the usage of Amazon Macie for scanning S3 buckets to identify and report Personally Identifiable Information (PII). The project also sets up alerts through EventBridge and SNS for immediate notification.

The implementation described here is specific to the us-east-1 region. Please make sure to adjust the instructions according to your deployment region.

It's important to note that Macie is free for 30 days, and as long as you disable it before the trial ends, there will be no charges incurred.

## Example Data

To simulate the presence of PII, the following example data files are provided:

- `cc.txt`: Contains credit card information

	```python
	American Express
	5135725008183484 09/26
	CVE: 550

	American Express
	347965534580275 05/24
	CCV: 4758

	Mastercard
	5105105105105100
	Exp: 01/27
	Security code: 912
	```

- `employees.txt`: Includes employee details

	```python
	74323 Julie Field
	Lake Joshuamouth, OR 30055-3905
	1-196-191-4438x974
	53001 Paul Union
	New John, HI 94740
	Amanda Wells

	354-70-6172
	242 George Plaza
	East Lawrencefurt, VA 37287-7620
	GB73WAUS0628038988364
	587 Silva Village
	Pearsonburgh, NM 11616-7231
	LDNM1948227117807
	Brett Garza
	```

- `keys.txt`: Contains access credentials and keys

	```python
	AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
	AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
	AWS_SESSION_TOKEN=AQoDYXdzEPT//////////wEXAMPLEtc764bNrC9SAPBSM22wDOk4x4HIZ8j4FZTwdQWLWsKWHGBuFqwAeMicRXmxfpSPfIeoIYRqTflfKD8YUuwthAx7mSEI/qkPpKPi/kMcGdQrmGdeehM4IC1NtBmUpp2wUE8phUZampKsburEDy0KPkyQDYwT7WZ0wq5VSXDvp75YU9HFvlRd8Tx6q6fE8YQcHNVXAkiY9q6d+xo0rKwT38xVqr7ZD0u0iPPkUL64lIZbqBAz+scqKmlzm8FDrypNC9Yjc8fPOLn9FX9KSYvKTr4rvx3iSIlTJabIQwj2ICCR/oLxBA==
	github_key: c8a2f31d8daeb219f623f484f1d0fa73ae6b4b5a
	github_api_key: c8a2f31d8daeb219f623f484f1d0fa73ae6b4b5a
	github_secret: c8a2f31d8daeb219f623f484f1d0fa73ae6b4b5a
	```

- `plates.txt`: Includes custom data in the form of Australian license plates

	```python
	# Victoria
	1BE8BE
	ABC123
	DEF-456

	# New South Wales
	AO31BE
	AO-15-EB
	BU-60-UB

	# Queensland
	123ABC
	000ZZZ
	987-YXW
	```

## Instructions

### Stage 1 - Uploading Example Data to S3

1. Save the provided example data as separate text files on your local machine.
2. Access the S3 console using the following link: [https://s3.console.aws.amazon.com/s3/buckets](https://s3.console.aws.amazon.com/s3/buckets).
3. Click on "Create bucket" to create a new S3 bucket.
4. Choose a name for the bucket and select the appropriate region (ensure it matches the Macie setup region).
5. Leave all other options as default and click "Create bucket."

Note: The region chosen for the bucket must match the region configured for Macie.

6. Upload the example data files to the S3 bucket you just created. You can either upload them directly to the root of the bucket or organize them into subdirectories.

Optional: If you prefer to upload the files via the command line to random subdirectories, you can use the following AWS CLI command (ensure you have [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) set up):

	```bash
	$ ls -1 *.txt
	cc.txt
	employees.txt
	keys.txt
	plates.txt

	$ for x in $(ls -1 *.txt); do aws s3 cp ${x} s3://BUCKET_NAME/$RANDOM/$RANDOM/$RANDOM/${x}; done
	upload: ./cc.txt to s3://BUCKET_NAME/1340/7073/9841/cc.txt
	upload: ./employees.txt to s3://BUCKET_NAME/15772/21602/11992/employees.txt
	upload: ./keys.txt to s3://BUCKET_NAME/26039/27331/32227/keys.txt
	upload: ./plates.txt to s3://BUCKET_NAME/14373/22304/25075/plates.txt
	```

### Stage 2 - Enabling Amazon Macie

1. Access the Macie console using the following link: [https://us-east-1.console.aws.amazon.com/macie/home?region=us-east-1#getStarted](https://us-east-1.console.aws.amazon.com/macie/home?region=us-east-1#getStarted).
2. After clicking on "Getting started," the initial screen will provide an introduction to Macie, including the creation of a service-linked role for Macie's S3 scanning capabilities.
3. Click on "Enable Macie" to enable the service.
4. Allow a few minutes for Macie to become ready. Refresh the page until you see the Macie dashboard.

	![Untitled](images/Untitled.png)

5. In the "S3 Buckets" section, select the bucket you created earlier and added the files to. Then click on "Create job."

	![Untitled](images/Untitled1.jpg)

6. On the "Review S3 buckets" page, click "Next."

7. On the "Refine the scope" page, select "One-time job" and click "Next."

8. Leave the "Managed data identifier options" set to "All" and click "Next."

9. Since there are no "Custom data identifiers" at this stage, click "Next."

10. On the "Select allow lists" page, click "Next."

11. Set a suitable "Job name" and click "Next."

12. Verify all the settings and details, and then click "Confirm."

Note: The job may take up to 20 minutes to complete. Once it's finished, you can click "Show results" and then "Show findings."

![Untitled](images/Untitled2.jpg)

13. You should see that the Credit Card, Employee, and Financial files have been detected as containing PII.

## Stage 3 - SNS Setup

To set up SNS for your project, follow these steps:

1. Access the SNS console by navigating to [https://us-east-1.console.aws.amazon.com/sns/v3/home?region=us-east-1#/topics](https://us-east-1.console.aws.amazon.com/sns/v3/home?region=us-east-1#/topics).

2. Click on the "Create topic" button.

3. Set the "Type" to "Standard" and name the topic as "Macie-Alerts".

4. In the "Access policy" section, keep the method as "Basic".

5. Configure the "Define who can publish messages to the topic" option to allow only specified AWS accounts. Enter your account ID, which can be found in the top right corner of the screen.

6. Similarly, configure the "Define who can subscribe to this topic" option to allow only specified AWS accounts. Enter your account ID again.

Note: In a production environment, it is advisable to further restrict access to only the necessary resources for publishing to the topic. However, for this temporary setup example, limiting access to the AWS account is sufficient and secure.

7. Keep all other options as default and click on the "Create topic" button.

8. On the next page, click on the "Create subscription" button.

9. Under the "Topic ARN", choose the sns topic created, which is "Macie-Alerts"

9. Change the "Protocol" to "Email".

10. In the "Endpoint" field, enter your personal email address.

11. Click the "Create subscription" button.

12. Shortly after, you will receive a confirmation email. Click on the link provided in the email to confirm your subscription. This step ensures that SNS can deliver emails to your specified endpoint and helps prevent spam.

Side note: While writing this, it was observed that the confirmation email sometimes goes to the Spam folder in Gmail. Please check your Spam folder if you don't receive the confirmation email in your inbox.

Once confirmed, your subscription should be in the "Confirmed" state as shown below:

![Untitled](images/Untitled3.jpg)

## Stage 4 - EventBridge Setup

To set up EventBridge for your project, follow these steps:

1. Access the EventBridge console by navigating to [https://us-east-1.console.aws.amazon.com/events/home?region=us-east-1#/](https://us-east-1.console.aws.amazon.com/events/home?region=us-east-1#/).

2. Ensure that "EventBridge Rule" is selected, and click the "Create rule" button.

	![Untitled](images/Untitled4.png)

3. On the next page, set the "Name" as "macie-events" and keep all other options as default. Click "Next".

4. Scroll down to the "Event pattern" section and select "Use pattern form".

5. Change the "AWS Service" to "Macie" and the "Event Type" to "Macie Finding".

	![Untitled](images/Untitled5.png)

6. Click "Next".

7. Set the "Target Type" to "AWS service" and then "SNS". Select the SNS Topic that you created earlier.

	![Untitled](images/Untitled6.png)

8. Click "Next".

9. Optionally, add any desired tags on the tags page and then click "Next".

10. On the final page, click the "Create rule" button to complete the setup.

## Stage 5 - Adding a Custom Macie Data Identifier

To add a custom Macie data identifier, follow these steps:

1. Access the Macie console by navigating to [https://us-east-1.console.aws.amazon.com/macie/home?region=us-east-1#summary](https://us-east-1console.aws.amazon.com/macie/home?region=us-east-1#summary).
2. Go to the "Custom data identifiers" page and click on the "Create" button.
3. Set the Name of the custom data identifier as "LicencePlates".
4. Specify the Regular Expression as follows:

   ```python
   ([0-9][a-zA-Z][a-zA-Z]-?[0-9][a-zA-Z][a-zA-Z])|([a-zA-Z][a-zA-Z][a-zA-Z]-?[0-9][0-9][0-9])|([a-zA-Z][a-zA-Z]-?[0-9][0-9]-?[a-zA-Z][a-zA-Z])|([0-9][0-9][0-9]-?[a-zA-Z][a-zA-Z][a-zA-Z])|([0-9][0-9][0-9]-?[0-9][a-zA-Z][a-zA-Z])
   ```

   This regular expression captures the license plate formats present in the example file mentioned.
5. Leave all other settings as default and click on the "Submit" button.

## Stage 6 - Starting a New Job

To start a new Macie job, follow these steps:

1. Go to the Macie console by navigating to [https://us-east-1.console.aws.amazon.com/macie/home?region=us-east-1#summary](https://us-east-1.console.aws.amazon.com/macie/home?region=us-east-1#summary).
2. Proceed to the "S3 Buckets" section and select the bucket where you have created and added files.
3. Tick the checkbox next to the selected bucket and click on the "Create job" button.

   ![Untitled](images/Untitled7.jpg)

4. On the "Review S3 buckets" page, click on the "Next" button.
5. On the "Refine the scope" page, choose "One-time job" and click on the "Next" button.
6. Keep the "Managed data identifier options" set to "All" and click on the "Next" button.
7. On the "Custom data identifiers" page, select the "LicencePlates" identifier that you created and click on the "Next" button.

   ![Untitled](images/Untitled8.png)

8. Proceed to the "Select allow lists" page and click on the "Next" button.
9. Set the desired "Job name" and click on the "Next" button.
10. Review the job configuration to ensure everything is correct, then click on the "Confirm" button.
11. After a few minutes, you should receive email notifications containing JSON outputs of the Macie findings, including the custom data identifier we just created.

   ![Untitled](images/Untitled9.jpg)

12. The job may take up to 20 minutes to complete. Once finished, click on "Show results" and then on "Show findings".

   ![Untitled](images/Untitled10.png)

13. Click on the custom identifier finding to view the details. You should see that it has detected 9 license plates in the plates.txt file.

   ![Untitled](images/Untitled11.jpg)

## Stage 7 - Clean Up

Follow these steps to clean up the resources

used in the previous stages:

1. Access the SNS console by navigating to [https://us-east-1.console.aws.amazon.com/sns/v3/home?region=us-east-1#/topics](https://us-east-1.console.aws.amazon.com/sns/v3/home?region=us-east-1#/topics).
2. Select the relevant Topic and click on the "Delete" button.
3. Enter "delete me" in the confirmation field and click on the "Delete" button.
4. Proceed to the Subscriptions page, select your subscription, and click on the "Delete" button. Confirm the deletion by clicking "Delete" again.
5. Go to the Macie console by navigating to [https://us-east-1.console.aws.amazon.com/macie/home?region=us-east-1#summary](https://us-east-1.console.aws.amazon.com/macie/home?region=us-east-1#summary).
6. In the Settings section, scroll to the end of the page and click on the "Disable Macie" button.
7. Enter "Disable" in the confirmation box and click on the "Disable Macie" button.

   ![Untitled](images/Untitled12.png)

8. Access the EventBridge console by navigating to [https://us-east-1.console.aws.amazon.com/events/home?region=us-east-1#/rules](https://us-east-1.console.aws.amazon.com/events/home?region=us-east-1#/rules).
9. Go to the Rules page, select the rule created earlier, and click on the "Delete" button.

   ![Untitled](images/Untitled13.jpg)

10. Enter the name of the rule in the confirmation box and click on the "Delete" button.
11. Access the S3 console by navigating to [https://s3.console.aws.amazon.com/s3/buckets?region=ap-southeast-2&region=ap-southeast-2](https://s3.console.aws.amazon.com/s3/buckets?region=ap-southeast-2&region=ap-southeast-2).
12. Select your source bucket and click on the "Empty" button.
13. In the confirmation window, enter "*permanently delete"*, and click on the "Empty" button.
14. Next, select your source bucket, click on the "Delete" button, and enter the bucket name in the confirmation window. Finally, click on the "Delete" button to delete the bucket.
