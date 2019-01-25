# 0x4447-Product-S3-Email

This stack is for those that want a free way to receive and send emails for their custom domains. Ideal when it is not worth paying a on-line service to have an email that you are going to use sporadically, and could care less about all the features that comes with that payed service. Also, ideal for developer to have test emails.

> Just receive and send email with some skills.

# How to deploy

Click on this button.

# Manual work

When you deploy this stack not everything is going be done for you. One missing aspect is adding your domain that you want to receive your email to SES, and confirm that you own it. This is due to CloudFomariont not supporting 100% of what AWS has to offer. To add your domain do the following:

1. Go to the SES Dashboard.
1. On the left side menu click `Domains`.
1. Click the blue button `Verify a New Domain`.
1. In the modal type your domain and select `Generate DKIM Settings`.
1. In the next window you will have all the information needed to configure your domain.
1. Once done you have to wait some time for the domain to switch from a status of `pending verification` to `verified`.

After the domain is confirmed you will be able to send emails from any email address within your domain.

# What will be deployed

This stack takes advantage of AWS S3, AWS SES, AWS Lambda and the AWS Trigger system to tie everything together.

- **SES**: is responsible for receiving and sending emails.
- **S3 Bucket**: is used to store the messages.
- **Lambda**: is used to process the emails.

# How dose it work

**Receiving an email**:

1. An email comes to SES which will trigger a Lambda function.
1. Tis lambda function will sort the email based on the To and From field and store it in S3 under the `Inbox` folder.
1. The `Inbox` folder will trigger a Lambda function which will load the raw email, convert it in to a `.html` and `.txt` file. Store it along side the original message.

**Sending an email**:

1. You create a JSON file properly formatted (check the section bellow)
1. Save the file in the following path `TMP/email_out/json`
1. This action will trigger the `json_to_raw` Lambda which will generate a raw email, send out using SES and saving the raw message to `TMP/email_out/raw`
1. This will trigger the `copy` Lambda, which will copy the raw message to the `Sent` folder organized by the To and From field.
1. Which will trigger the the same `converter` Lambda used in the **Receiving an email** section to convert the raw message in readable files.

This flow was designed to take only advantage of the S3 trigger system, and brake each action in to small Lambdas. One limiting factor of the trigger system is that a PUT action is indistinguishable from each other. Which can causes infitnie loops if. To mitigate this you either have to wrtie extra code to check which object triggered a Lambda, and from where. Or you can differentiate an action by doing a COPY action. This ads an extra step in the flow, but removes the need of extra code. Both solutions are valid, and in this case we decided to opt for the PUT/COPY actions and see how this works.

# How to make the email message

You create a custom JSON file which you then upload to the `TMP/email_out/json` folder, and the file should look like this:

```
{
	"from": "name@example.com",
	"to": "name@example.com",
	"subject": "From SES",
	"html": "Write a nice message to whoever you are sending this message to.",
	"text": "Write a nice message to whoever you are sending this message to."
}
```

Remember that the `from` field must use the domain that you added to SES, you won't be able to send email from domains that you did not verified.

# SES Limitation

There are to major limitation with SES:

1. For security reasons AWS by default allows you to send 200 emails per 24 hour period with a rate of 1 email/second. If you think you'll send more then that, you'll have to ask AWS to increase your limit.
1. By default you can't send emails to unverified addresses. If you want the ability to send out and not only receive, you'll have to reach out to AWS to remove this limitation from your account.

# Pricing

All the resources deployed with this stack will potentially cost you money. But for this to happen you'll have to:

- Invoke your lambdas over 1,000,000 times a month.
- Send and receive over 1000 emails a month.
- Perform over 10000 Get and Put operations, and over 2000 Delete ones in your S3 Bucket.

The only payment that you'll encounter from day one, is S3 storage for the emails that you receive and create.

# How to work with this project

If you'd like to change some aspects of this stack, don't edit the `CloudFormation.json` file. We recommend you use the [Grapes framework](https://github.com/0x4447/0x4447-cli-node-grapes). This tool was designed to make it easier to work with Cloud Formation file. You should never edit the main CF file.

# The End

If you enjoyed this project, please consider giving it a 🌟. And check out our [0x4447 GitHub account](https://github.com/0x4447), where we have additional resources that you might find useful or interesting.

## Sponsor 🎊

This project is brought to you by 0x4447 LLC, a software company specializing in build custom solutions on top of AWS. Find out more by following this link: https://0x4447.com or, send an email at [hello@0x4447.email](mailto:hello@0x4447.email?Subject=Hello%20From%20Repo&Body=Hi%2C%0A%0AMy%20name%20is%20NAME%2C%20and%20I%27d%20like%20to%20get%20in%20touch%20with%20someone%20at%200x4447.%0A%0AI%27d%20like%20to%20discuss%20the%20following%20topics%3A%0A%0A-%20LIST_OF_TOPICS_TO_DISCUSS%0A%0ASome%20useful%20information%3A%0A%0A-%20My%20full%20name%20is%3A%20FIRST_NAME%20LAST_NAME%0A-%20My%20time%20zone%20is%3A%20TIME_ZONE%0A-%20My%20working%20hours%20are%20from%3A%20TIME%20till%20TIME%0A-%20My%20company%20name%20is%3A%20COMPANY%20NAME%0A-%20My%20company%20website%20is%3A%20https%3A%2F%2F%0A%0ABest%20regards.).