# 0x4447-Product-S3-Email

This stack is for those that want a free way to receive and send emails for their custom domains. Ideal when it is not worth paying a on-line service to have an email that you are going to use sporadically, could care less about all the added features. Ideal for:

- individuals that have some technical skills,
- test emails for developer,
- or organizations that want to give emails to thousands of employees at scale (they need to build their own UI).

> Just receive and send email with some skills.

# DISCLAIMER!

This stack is available to anyone at no cost, but on an as-is basis. 0x4447 LLC is not responsible for damages or costs of any kind that may occur when this stack is used. You take full responsibility when you use it.

# How to deploy

Click on this button.

# Manual work

When you deploy this stack not everything is going be done for you.

### Confirm you own the domain

To add your domain do the following:

1. Go to the SES Dashboard.
1. On the left side menu click `Domains`.
1. Click the blue button `Verify a New Domain`.
1. In the modal type your domain and select `Generate DKIM Settings`.
1. In the next window you will have all the information needed to configure your domain.
1. Once done you have to wait some time for the domain to switch from a status of `pending verification` to `verified`.

After the domain is confirmed you will be able to send emails from any email address within your domain.

### Enable SES Rule Sets

The deployment will create a SES rules set which should be enabled by default, but because of a know bug in CloudFormation, this dose not always happen. To enable the rule, do the following:

1. Go to the SES Dashboard.
1. On the left side menu click `Rule Sets`.
1. From the `Inactive Rule Sets` tab check the `0x4447_S3_Email`
1. Then hit `Set as Active Rule Set` which will activate the rule.

Once this is done, you will be able to process incoming emails.

# What will be deployed

![S3-Email Diagram](https://raw.githubusercontent.com/0x4447/0x4447-product-s3-email/assets/diagram.png)

This stack takes advantage of AWS S3, AWS SES, AWS Lambda and the AWS Trigger system to tie everything together.

- 1x SES Rule Sets
- 2x S3 Bucket
  - 1x for CodePipeline to store artifacts
  - 1x for the emails
- 3x CodePipelines
- 6x CodeBuilds
- 3x Lambdas

Once deployed the code for the Lambdas will be automatically updated, and the CodePipeline will keep the Lambda code up-to-date since it will push new code every time the selected brunch gets new code.

All the resources of the project can be found [here](https://github.com/0x4447?utf8=%E2%9C%93&q=0x4447-product-s3-email).

# How dose it work

**Receiving an email**:

1. An email comes to SES which will trigger a Lambda function.
1. This lambda function will sort the email based on the To and From field and store it in S3 under the `Inbox` folder.
1. The `Inbox` folder will trigger a Lambda function which will load the raw email, convert it in to a `.html` and `.txt` file, and store it along side the original message.

**Sending an email**:

1. You create a JSON file properly formatted (check the section bellow)
1. Save the file in the following path `TMP/email_out/json`. The file name or extension are irrelevant as long as the content is text and JSON formated.
1. This action will trigger the `outbound` Lambda which will generate a raw email, send it out using SES and save the raw message to the `Sent` folder.
1. The `Sent` folder will trigger a Lambda function which will load the raw email, convert it in to a `.html` and `.txt` file, and store it along side the original message.

This flow was designed to take only advantage of the S3 trigger system, and brake each action in to small Lambda.

### How to make the email message

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

Remember that the `from` field must use the domain that you added to SES, you won't be able to send emails from domains that you did not verify.

# SES Limitation

There are two major limitation with SES:

1. For security reasons AWS by default allows you to send 200 emails per 24 hour period with a rate of 1 email/second. If you think you'll send more then that, you'll have to ask AWS to increase your limit.
1. By default you can't send emails to unverified addresses. If you want the ability to send out (not only receive), you'll have to reach out to AWS to remove this limitation from your account.

# Pricing

All the resources deployed with this stack will potentially cost you money. But for this to happen you'll have to:

- Invoke your lambdas over 1,000,000 times a month.
- Send and receive over 1000 emails a month.
- Perform over 10000 Get and Put operations, and over 2000 Delete ones in your S3 Bucket.
- Exceed 100 build minutes on CodeBuild
- $1 per active CodePipeline. A CodePipeline needs to run at least once a month to be considered active.

The only payment that you'll encounter from day one, is S3 storage for the emails and CodePipeline artifacts.

# How to work with this project

If you'd like to change some aspects of this stack, don't edit the `CloudFormation.json` file. We recommend you use the [Grapes framework](https://github.com/0x4447/0x4447-cli-node-grapes). This tool was designed to make it easier to work with Cloud Formation file. You should never edit the main CF file.

# The End

If you enjoyed this project, please consider giving it a 🌟. And check out our [0x4447 GitHub account](https://github.com/0x4447), where we have additional resources that you might find useful or interesting.

## Sponsor 🎊

This project is brought to you by 0x4447 LLC, a software company specializing in building custom solutions on top of AWS. Follow this link to learn more: https://0x4447.com. Alternatively, send an email to [hello@0x4447.email](mailto:hello@0x4447.email?Subject=Hello%20From%20Repo&Body=Hi%2C%0A%0AMy%20name%20is%20NAME%2C%20and%20I%27d%20like%20to%20get%20in%20touch%20with%20someone%20at%200x4447.%0A%0AI%27d%20like%20to%20discuss%20the%20following%20topics%3A%0A%0A-%20LIST_OF_TOPICS_TO_DISCUSS%0A%0ASome%20useful%20information%3A%0A%0A-%20My%20full%20name%20is%3A%20FIRST_NAME%20LAST_NAME%0A-%20My%20time%20zone%20is%3A%20TIME_ZONE%0A-%20My%20working%20hours%20are%20from%3A%20TIME%20till%20TIME%0A-%20My%20company%20name%20is%3A%20COMPANY%20NAME%0A-%20My%20company%20website%20is%3A%20https%3A%2F%2F%0A%0ABest%20regards.).