# Implementing a Serverless Architecture with AWS Managed Services

## Steps
- upload an inventory file to an S3 bucket
- this will trigger an aws lambda function that will read the file and insert items into an Amazon DynamoDB table
- a serverless web-based dashboard application will use Amazon Cognito to authenticate to AWS and then gain access to the dynamoDB table to display inventory levels
- another aws lambda function will receive updates from the DynamoDB table and will send a message to an amazon Simple Notification Service (SNS) topic when an inventory item is out of stock
- Amazon SNS will then send an SMS or emal notification to you to request additional inventory

1. Create a Lambda Function to Load Data
    - from the lambda console, create a function (Author from Scratch), name, runtime py 3.7, (you need a lambda role for this that gives execution permissions to the lambda function so it can access S3 and DynamoDB) and the function is the following:
        ```
        # Load-Inventory Lambda function
        #
        # This function is triggered by an object being created in an Amazon S3 bucket.
        # The file is downloaded and each line is inserted into a DynamoDB table.

        import json, urllib, boto3, csv

        # Connect to S3 and DynamoDB
        s3 = boto3.resource('s3')
        dynamodb = boto3.resource('dynamodb')

        # Connect to the DynamoDB tables
        inventoryTable = dynamodb.Table('Inventory');

        # This handler is executed every time the Lambda function is triggered
        def lambda_handler(event, context):

        # Show the incoming event in the debug log
        print("Event received by Lambda function: " + json.dumps(event, indent=2))

        # Get the bucket and object key from the Event
        bucket = event['Records'][0]['s3']['bucket']['name']
        key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'])
        localFilename = '/tmp/inventory.txt'

        # Download the file from S3 to the local filesystem
        try:
            s3.meta.client.download_file(bucket, key, localFilename)
        except Exception as e:
            print(e)
            print('Error getting object {} from bucket {}. Make sure they exist and your bucket is in the same region as this function.'.format(key, bucket))
            raise e

        # Read the Inventory CSV file
        with open(localFilename) as csvfile:
            reader = csv.DictReader(csvfile, delimiter=',')

            # Read each row in the file
            rowCount = 0
            for row in reader:
            rowCount += 1

            # Show the row in the debug log
            print(row['store'], row['item'], row['count'])

            try:
                # Insert Store, Item and Count into the Inventory table
                inventoryTable.put_item(
                Item={
                    'Store':  row['store'],
                    'Item':   row['item'],
                    'Count':  int(row['count'])})

            except Exception as e:
                print(e)
                print("Unable to insert data into DynamoDB table".format(e))

            # Finished!
            return "%d counts inserted" % rowCount
            ```
        > the code downloads the file from S3 that triggered the event, loops through each line in the file, and inserts the data into the DynamoDB inventory table
    2. Configure an S3 Event
        - create an S3 bucket from the S3 console, and once created navigate to the properties tab. From advanced settings, click events. Add notification and configure name, Events (check all object create events), send to lambda function, select lambda function created in step 1 and save!
    3. upload a csv file to s3 to test the lambda function uploaded data int the dynamo db table!
    4. Configure notifications
        - from the SNS console, create a topic and configure topic name, display name, and protocol sms (with your personal phone number). this example will send an sms notification when inventory for a specific item is out of stock.
    5. Create a lambda function to send notifications
        - this lambda function will trigger whenever data is loaded into the DynamoDB table, triggered by a DynamoDB stream. If it notices an item is Out of Stock, it will send a notification via the SNS topic crated in step 4.
        > benefits to this architectural approach of having dedicated lambda functions:
            - each lambda function performs a single, specific function. this makes code simpler & more maintainable
            - addiional business logic can be added by creating additional lambda functions. each function operates independently so existing functionality is not impacted
        - create a lambda function with configurations name, runtime py3.7, a lambda role that has permissions to send a notification to SNS and the following code: 
        ```
        # Stock Check Lambda function
        #
        # This function is triggered when values are inserted into the Inventory DynamoDB table.
        # Inventory counts are checked and if an item is out of stock, a notification is sent to an SNS Topic.

        import json, boto3

        # This handler is executed every time the Lambda function is triggered
        def lambda_handler(event, context):

        # Show the incoming event in the debug log
        print("Event received by Lambda function: " + json.dumps(event, indent=2))

        # For each inventory item added, check if the count is zero
        for record in event['Records']:
            newImage = record['dynamodb'].get('NewImage', None)
            if newImage:

            count = int(record['dynamodb']['NewImage']['Count']['N'])

            if count == 0:
                store = record['dynamodb']['NewImage']['Store']['S']
                item  = record['dynamodb']['NewImage']['Item']['S']

                # Construct message to be sent
                message = store + ' is out of stock of ' + item
                print(message)

                # Connect to SNS
                sns = boto3.client('sns')
                alertTopic = 'NoStock'
                snsTopicArn = [t['TopicArn'] for t in sns.list_topics()['Topics']
                                if t['TopicArn'].lower().endswith(':' + alertTopic.lower())][0]

                # Send message to SNS
                sns.publish(
                TopicArn=snsTopicArn,
                Message=message,
                Subject='Inventory Alert!',
                MessageStructure='raw'
                )

        # Finished!
        return 'Successfully processed {} records.'.format(len(event['Records']))
        ```
    6. Test SMS capability by uploading another file to S3!
## defs
    - SNS is a flexible, fully managed publish/subscribe messaging and mobile notifications service for the delivery of messages to subscribing enpoints and clients. With SNS you can fan-out messages to a large number of subscribers, including distributed systems and services, and mobile devices.