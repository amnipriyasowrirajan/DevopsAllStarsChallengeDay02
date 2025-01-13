# Game Notification App 📧🏀

This project automates the process of sending NBA game score updates directly to your email using **AWS SNS**, **Lambda**, and **EventBridge**. Stay updated with real-time notifications for every game, without lifting a finger!

---

## Features 🚀

- **AWS SNS**: Sends email notifications with game scores.
- **AWS Lambda**: Fetches game data from the NBA API and triggers notifications.
- **AWS EventBridge**: Schedules notifications at regular intervals.
- **SportsData.io API**: Provides real-time NBA game scores.

---

## Architecture Diagram 🏗️

![Diagram](<Screenshot 2025-01-11 115252.png>)

---

## Prerequisites ✅

1. **AWS Account**: To access AWS services.
2. **SportsData.io API Key**: Get a free API key [here](https://sportsdata.io/).
3. **Python 3.10**: For Lambda function.

---

## Setup Instructions ⚙️

### Step 1: Create an SNS Topic

1. Open the **AWS SNS Console**.
2. Create a **Standard Topic** named `gd_topic`.
3. Note the **ARN** of the topic.

### Step 2: Create an Email Subscription

1. Add an email subscription to the SNS topic.
2. Confirm the subscription via the email link.
3. Verify the subscription status in the SNS console.

### Step 3: Create an IAM Role

1. Go to **IAM Roles** and create a role:
   - **Service:** Lambda
   - Attach the following policies:
     - `AWSLambdaBasicExecutionRole`
     - A custom policy allowing `sns:Publish` to your topic:
       ```json
       {
         "Version": "2012-10-17",
         "Statement": [
           {
             "Effect": "Allow",
             "Action": "sns:Publish",
             "Resource": "arn:aws:sns:REGION:ACCOUNT_ID:gd_topic"
           }
         ]
       }
       ```
   - Replace `REGION` and `ACCOUNT_ID` with your AWS details.
2. Name the role `gd_lambda_role`.

### Step 4: Create a Lambda Function

1. Navigate to **AWS Lambda Console**.
2. Create a function:
   - **Name:** `gd_notifications`
   - **Runtime:** Python 3.10
   - **Role:** Select `gd_lambda_role`.
3. Add the following environment variables:

   - `NBA_API_KEY`: Your API key from SportsData.io.
   - `SNS_TOPIC_ARN`: ARN of the SNS topic.

4. Write the Lambda function (`lambda_function.py`) and deploy it:

   ```python
   import boto3
   import requests
   import os

   def lambda_handler(event, context):
       sns_client = boto3.client('sns')
       api_key = os.getenv('NBA_API_KEY')
       sns_topic_arn = os.getenv('SNS_TOPIC_ARN')

       # Fetch NBA game data
       url = f"https://api.sportsdata.io/v4/nba/scores/json/GamesByDate/YOUR_DATE"
       headers = {"Ocp-Apim-Subscription-Key": api_key}
       response = requests.get(url, headers=headers)
       games = response.json()

       # Prepare message
       message = "NBA Game Scores:\n"
       for game in games:

### Step 5: Automate with EventBridge
   Open the EventBridge Console.
   Create a rule:
   Name: gd_rule
   Schedule Type: Cron-based.
   Cron Expression: 0 9-23/2,0-2 * * ? * (Every two hours from 9 AM to 2 AM).
   Attach the rule to the gd_notifications Lambda function.
### Step 6: Test the App
   Test the Lambda function in the AWS console.
   Check your email for game score updates.
   Monitor scheduled notifications via EventBridge.
           message += f"{game['HomeTeam']} vs {game['AwayTeam']} - {game['HomeTeamScore']}:{game['AwayTeamScore']}\n"

### Publish to SNS
   sns_client.publish(TopicArn=sns_topic_arn, Message=message)
   return {"statusCode": 200, "body": "Notification sent!"}
   ```
