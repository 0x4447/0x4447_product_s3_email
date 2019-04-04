# 0x4447-Product-S3-Email

This stack was created out of frustration over the complicated process of creating and running a full email server. Even today, we're still dealing with the overhead of installing and configuring multiple necessary servers to handle incoming and outgoing messages. We wanted something simple - with no interface and no server management - so we came up with S3-Email, which includes AWS SES as our email server (receive and send) and S3 as our database and interface. Then we tied everything together with a bit of code via AWS Lambda.

The result is an unmanaged email server offering unlimited email addresses and the benefit of easily organizing messages by adding the `+` character to the email names. The `+` is converted to a `/` that correlates to an object path in S3.

### Organizing with a +

When you sign up for online services, use the `+` character to organize your emails as follows:

- social+facebook@example.com
- social+instagram@example.com
- social+linkedin@example.com
- and so on...

This groups all social emails in the `social` folder. The possibilities are endless.

### Endless email addresses

Once you've added and confirmed your domain with SES, you can put any string that conforms with the email address standard in front of the `@`. S3-Email puts endless potential email addresses at your disposal. It also provides you with an opportunity to organize your life in a way that hasn't been possible until now. For example, you can assign each service you use its own dedicated email:

- facebook@example.com
- instagram@example.com
- linkedin@example.com
- etc.

> Basically, you can receive and send email with some skills.

# DISCLAIMER!

This stack is available to everyone at no cost, but on an as-is only basis. 0x4447 LLC is not responsible for damages or costs of any kind that may occur when you use the stack. You take full responsibility when you use it.

# How to deploy

<a target="_blank" href="https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=zer0x4447-S3-Email&templateURL=https://s3.amazonaws.com/0x4447-drive-cloudformation/s3-email.json">
<img align="left" style="float: left; margin: 0 10px 0 0;" src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png"></a>

All you need to do to deploy the stack is click the button at left and follow the instructions that CloudFormation provides in your AWS Dashboard. Alternatively, you can download the CF file from [here](https://s3.amazonaws.com/0x4447-drive-cloudformation/s3-email.json).

# What deploys?

![S3-Email Diagram](https://raw.githubusercontent.com/0x4447/0x4447-product-s3-email/assets/diagram.png)

The stack takes advantage of AWS S3, AWS SES, AWS Lambda, and the AWS Trigger system to tie everything together. You'll have the following:

- 1x SES Rule Sets
- 2x S3 Bucket
  - 1x for CodePipeline to store artifacts
  - 1x for emails
- 3x CodePipelines
- 3x CodeBuilds
- 3x Lambdas

All project resources can be found [here](https://github.com/topics/0x4447-product-s3-email).

# Auto deploy

The stack is set up in a such a way that any time new code is pushed to a selected branch, the CodePipeline picks up the change and updates the Lambda for you. These are the available branches:

- **master**: the latest stable code
- **development**: unstable code that we test in our test environment - we don't recommend that you use this branch

# Manual work

Keep in mind that when you deploy, everything may not work right out of the box.

### Confirm that you own the domain

Follow these steps to add your domain and confirm that you own it.

1. Go to the SES Dashboard.
2. Click `Domains` on the left side menu.
3. Click the blue `Verify a New Domain` button.
4. Type your domain in the modal and select `Generate DKIM Settings`.
5. View all the information you need to configure your domain in the next window displayed.
6. Wait for a period of time for the domain status to switch from `pending verification` to a `verified`.

### Enable SES Rule Sets

Deployment creates SES `rule sets`. This should be enabled by default, but it doesn't always happen because of a known bug in CloudFormation. In that case, take the following steps to enable the rule:

1. Go to the SES Dashboard.
2. Click `Rule Sets` on the left side menu.
3. Check `0x4447_S3_Email` on the `Inactive Rule Sets` tab.
4. Hit `Set as Active Rule Set` to activate the rule.

# SES Limitations

SES comes with the following major limitations:

1. For security reasons, AWS defaults to 200 emails sent per 24-hour period at a rate of 1 email per second. If you need to send more than that, you'll need to ask AWS to increase your limit.
2. By default, you can't send emails to unverified addresses. If you'd like to be able to send (as opposed to just receiving), reach out to AWS to remove this limitation from your account.

# How the stack works

**Receiving email**:

1. An email comes to SES and and is stored in the `TMP` S3 folder.
2. S3 triggers the Inbound Lambda Function, which then organizes the email based on the `to`, `from` and `date` fields. In addition, the Lambda reads the domain added to SES and uses it to determine whether the email should land in the `Inbox` or `Sent` folder. If the `to` field contains the domain from SES, it goes to the `Inbox`. If not, it assumes that the email was sent out.
3. The `Inbox` or `Sent` folder triggers another Lambda function that loads the raw email, converts it to a `.html` and `.txt` file, and stores it alongside the original message.

**Sending email**:

1. Create a properly formatted JSON file (see the following section).
2. Save the file to the path `TMP/email_out/json`. The file name and extension are irrelevant as long as the content is text and JSON formatted.
3. This action triggers a Lambda that generates a raw email, sends it out using SES, and saves the raw message to the `Sent` folder.
4. The `Sent` folder triggers another Lambda function that loads the raw email, converts it to `.html` and `.txt` files, and stores them alongside the original message.

This flow was designed to take advantage of the S3 trigger system and break each action into a small Lambda.

### How to create an email message

Create a custom JSON file, then upload it to the `TMP/email_out/json` folder (if you haven't structured the folder, set it up now). The JSON structure should look like this:

```
{
	"from": "name@example.com",
	"to": "name@example.com",
	"subject": "From SES",
	"html": "Write a nice message to whoever you are sending this message to.",
	"text": "Write a nice message to whoever you are sending this message to."
}
```

Remember that the `from` field must contain the domain you previously added to SES. You won't be able to send emails from unverified domains.

# Backup old emails

Bonus feature: Thanks to the stack's event-driven nature, it's possible to process manually uploaded emails. Simply upload the raw email to the `TMP/email_in` folder, and your emails will be processed automatically.

The only prerequisite is that the email file has to be the raw representation of the email itself. For example, files ending in `.eml` are nothing more than txt files containing the raw content of the email.

This means you can just directly upload those files to the S3 bucket.

# Pricing

While all resources deployed via this stack could potentially cost you money, you'd have to do the following for that to happen:

- Invoke over 1,000,000 Lambdas per month
- Send and receive over 1000 emails per month
- Perform over 10,000 Get and Put operations and over 2000 Delete operations in your S3 Bucket
- Exceed 100 build minutes on CodeBuild
- Run at least one active CodePipeline per month at $1 (must run at least once a month to be considered active)

The only payment you'll encounter from Day One is an S3 storage fee for emails and CodePipeline artifacts.

# How to work with this project

When you want to deploy the stack, the `CloudFormation.json` file is the only file you should be interested in. If you'd like to modify the stack, we recommend using the [Grapes framework](https://github.com/0x4447/0x4447-cli-node-grapes), which we designed to make it easier to work with the CloudFormation file. If you'd like to keep your sanity, never edit the main CF file 🤪.

# The End

If you enjoyed this project, please consider giving it a 🌟. And be sure to check out our [0x4447 GitHub account](https://github.com/0x4447), which contains additional resources that you might find useful or interesting.

## Sponsor 🎊

This project is brought to you by 0x4447 LLC, a software company specializing in building custom solutions on top of AWS. Follow this link to learn more: https://0x4447.com. Alternatively, send an email to [hello@0x4447.email](mailto:hello@0x4447.email?Subject=Hello%20From%20Repo&Body=Hi%2C%0A%0AMy%20name%20is%20NAME%2C%20and%20I%27d%20like%20to%20get%20in%20touch%20with%20someone%20at%200x4447.%0A%0AI%27d%20like%20to%20discuss%20the%20following%20topics%3A%0A%0A-%20LIST_OF_TOPICS_TO_DISCUSS%0A%0ASome%20useful%20information%3A%0A%0A-%20My%20full%20name%20is%3A%20FIRST_NAME%20LAST_NAME%0A-%20My%20time%20zone%20is%3A%20TIME_ZONE%0A-%20My%20working%20hours%20are%20from%3A%20TIME%20till%20TIME%0A-%20My%20company%20name%20is%3A%20COMPANY%20NAME%0A-%20My%20company%20website%20is%3A%20https%3A%2F%2F%0A%0ABest%20regards.).
