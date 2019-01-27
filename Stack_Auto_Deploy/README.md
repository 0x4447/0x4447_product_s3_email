# CodeBuild for GitHub Project

This CloudFormation file will help you setup a CodeBuild that will monitor whatever branch you specify, and will updated your website that is being delivered through loudFront.

# DISCLAIMER!

This stacks are available to anyone at no cost, but on an as-is basis. 0x4447 LLC is not responsible for damages or costs of any kind that may occur when this stack is used. You take full responsibility when you use them.

# How to deploy

Click on this button.

# What will be deployed

- 1x CodeBuild
- 1x Role

# How to setup your repo

1. Make your website with any framework. If you need a simple one, you can use our [Avocado](https://www.npmjs.com/package/@0x4447/avocado).
1. Create a `buildspec.yml` file inside the root directory of your project. You can take inspiration from our [company website](https://github.com/0x4447/0x4447.com) one.

# How to work with this project

If you'd like to change some aspects of this stack, don't edit the `CloudFormation.json` file. We recommend you use the [Grapes framework](https://github.com/0x4447/0x4447-cli-node-grapes). This tool was designed to make it easier to work with Cloud Formation file. You should never edit the main CF file.

# The End

If you enjoyed this project, please consider giving it a 🌟. And check out our [0x4447 GitHub account](https://github.com/0x4447), where we have additional resources that you might find useful or interesting.

## Sponsor 🎊

This project is brought to you by 0x4447 LLC, a software company specializing in building custom solutions on top of AWS. Follow this link to learn more: https://0x4447.com. Alternatively, send an email to [hello@0x4447.email](mailto:hello@0x4447.email?Subject=Hello%20From%20Repo&Body=Hi%2C%0A%0AMy%20name%20is%20NAME%2C%20and%20I%27d%20like%20to%20get%20in%20touch%20with%20someone%20at%200x4447.%0A%0AI%27d%20like%20to%20discuss%20the%20following%20topics%3A%0A%0A-%20LIST_OF_TOPICS_TO_DISCUSS%0A%0ASome%20useful%20information%3A%0A%0A-%20My%20full%20name%20is%3A%20FIRST_NAME%20LAST_NAME%0A-%20My%20time%20zone%20is%3A%20TIME_ZONE%0A-%20My%20working%20hours%20are%20from%3A%20TIME%20till%20TIME%0A-%20My%20company%20name%20is%3A%20COMPANY%20NAME%0A-%20My%20company%20website%20is%3A%20https%3A%2F%2F%0A%0ABest%20regards.).