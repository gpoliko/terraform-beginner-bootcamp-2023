# Terraform Beginner Bootcamp 2023

## Semantic Versioning :mage:
This project is going utilize semantic versioning for its tagging. [semver.org](https://semver.org/)

The general format:

**MAJOR.MINOR.PATCH**, eg. `1.0.1`

- **MAJOR** version when you make incompatible API changes
- **MINOR** version when you add functionality in a backward compatible manner
- **PATCH** version when you make backward compatible bug fixes

## Install the Terraform CLI

### Considerations with the Terrafor CLI Changes
The Terraform CLI installation instructions have changed due to gpg keypring changes. So we needed to refer to the latest install CLI instructions via Terraform Documentation and change the scripting for install.

[Install Terraform CLI](https://www.developer.hasicorp.com/terraform/tutorials/aws-get-started/install-cli)

Example of checking OS Version:
```
$ cat /etc/os-release
PRETTY_NAME="Ubuntu 22.04.3 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.3 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```

### Considerations for Linux Distribution

This project is built against Ubuntu.
Please consider checking your Linux distribution and change accordingly to your distribution needs.
[How To Check OS Version In Linux](https://www.cyberciti.biz/faq/how-to-check-os-version-in-linux-command-line/)

### Refactoring into Bash Scripts

While fixing the Terraform CLI gpg deprecation issues we noticed that the bash script steps were a considerable amount more code. So we decided to create a bash script to install the Terraform CLI.

This bash script is located here: [./bin/install_terraform_cli](./bin/install_terraform_cli)

- This will keep the Gitpod Task File ([.gitpod.yml](.gitpod.yml)) tidy.
- This allows us an easier to debug and execute manually Terraform CLI install.
- This will allow better portability for other projects that need to insall Terraform CLI.

#### Shebang Considerations
A Shebang (prounced Sha-bang) tells the bash script what program that will interpret the script. eg. `#!/bin/bash`

ChatGPT recommended this format for bash: `#!/usr/bin/env bash`

- For portability for different OS distributions
- Will search the user's PATH for the bash executable

https://en.wikipedia.org/wiki/Shebang_(Unix)

#### Execution Considerations

When executing the bash script we can use the `./` shorthand notation to execute the bash script.

eg. `./bin/install_terraform_cli`

If we are using a script in gitpod.yml, we need to point the script to a program to interpret it.

eg.`source ./bin/install_terraform_cli`

#### Linux Permissions Considerations

In order to make our bash scripts executable we need to change the Linux permission for the fix to be executable at the user mode.

```sh
chmod u+x ./bin/install_terraform_cli
```

Alternatively:

```sh
chmod 744 ./bin/install_terraform_cli
```
https://en.wikipedia.org/wiki/Chmod

### Gitpod Lifecycle (Before, Init, Command)

We need to be careful when using the Init, because it will not rerun if we restart an existing workspace.

https://www.gitpod.io/docs/configure/workspaces/tasks


### Working with Env Vars

#### env command

We can list out all Environment Variables (Env vars) usign the `env` command.

We can filter specific env vars using grep eg. `env | grep AWS_`

#### Setting and Unsetting Env Vars

In the terminal we can set using `export HELLO='world'`

In the terminal we unset using `unset HELLO`

We can set an env var temporarily when we just run a command.
```sh
HELLO='world' ./bin/print_message
```
Within a bash script we can set env without writing export eg.
```sh
#!/usr/bin/env bash

HELLO='world'

echo $HELLO
```

#### Printing Vars

We can print an env var using echo eg. `echo $HELLO`

#### Scoping of Env Vars

When you open up new bash terminals in VSCode, it will not be aware of env vars that you have set in another window.

If you want Env Vars to persist across all future bash terminals that you open, you need to set env vars in your bash profile. eg. `.bash_profile`

#### Persisting Env Vars in Gitpod

> **NOTE**: Because I have set my Github email to private, Gitpod cannot determine what my email is for committing and pushing to remote.
Which means that I have to set the global git email and Gitpod env var to my private email every time I launch Gitpod.
***To get around this I have created a bash script, [bin/set_global_git_email](/bin/set_global_git_email) to automate setting the email for me.***

We can persist env vars into gitpod by storing them in Gitpod Secrets Storage.
```
gp env HELLO='world'
```

All future workspaces launched will set the env vars for all bash terminals openend in those workspaces.

You can also set env vars in the `.gitpod.yml` but this can only contain non-sensitive env vars.

### AWC CLI Installation

AWS CLI is installed for the project via the bash script [`./bin/install_aws_cli`](./bin/install_aws_cli)

[Getting Started Install (AWS CLI)](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
[AWS CLI Env Vars](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html)

We can check if our AWS credentials are configured correctly, by running the following cli command:
```sh
aws sts get-caller-identity
```

If it is successful you should see a JSON payload return that looks like this:
```json
{
    "UserId": "4IRA2KXJX8T247NSJRUSR",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/terraform-beginner-bootcamp-2023"
}
```

We'll need to generate AWS CLI credentials from IAM User in order to use AWS CLI.


## Terraform Basics

### Terraform Registry

Terraform sources their providers and modules from the Terraform registry which is located at registry [registry.terraform.io](https://registry.terraform.io/)

- **Provider** is an interface to APIs that will allow you to create resources in Terraform.
- **Modules** are a way to make large amounts of Terraform code modular, portable, and sharable.

[Random Terraform Provider](https://registry.terraform.io/providers/hashicorp/random)

### Terraform Console

We can see a list of all the Terraform commands by simply typing `terraform`

#### Terraform Init

`terraform init`

At the start of a new Terraform project we will run `terraform init` to download the binaries for the Terraform providers that we'll use in this project.

#### Terraform Plan

`terraform plan`

This will generate out a changeset about the state of our infrastructure and what will be changed.

We can output this changeset ie. "plan" to be passed to an apply, but often you can just ignore outputting.

#### Terraform Apply

`terraform apply`

This will run a plan and pass the changeset to be executed by Terraform. Apply should prompt us yes or no.

If we want to automatically approve an apply we can provide auto approve flag eg. `terraform apply --auto-approve`.

### Terraform Destroy

`terraform destroy`
This will destroy resources

You can also use the auto approve flag to skip the approve prompt
e.g. `terraform apply --auto-approve`

### Terraform Lock Files

`.terraform.lock.hcl` contains the locked versioning for the providers or modules that should be used with this project.

The Terraform Lock File **should be commited** to your Version Control System (VSC) eg. Github.

### Terraform State Files

`.terraform.tfstate` contains information about the current state of your infrastructure.

This file **should not be committed** to your VCS.

This file can contain sensitive data.

If you lose this file, you lose knowing the state of your infrastructure.

`.terraform.tfstate.backup` is the previous state.

### Terraform Directory

`.terraform` directory contains binaries of Terraform providers.

### Issues with Terraform Cloud Login and Gitpod Workspace

When attempting to run `terraform login` it will launch in bash a wiswig view to generate a token. However, it does not work as expected in Gitpod VSCode in the browser.

The work around is to manually generate a token in Terraform Cloud.
```
https://app.terraform.io/app/settings/tokens?source=terraform-login
```

Then create the file manually here:
```sh
touch /home/gitpod/.terraform.d/credentials.tfrc.json
open /home/gitpod/.terraform.d/credentials.tfrc.json
```

Provide the following code (replace your token in the file):

```json
{
    "credentials": {
        "app.terraform.io": {
            "token": "YOUR-TERRAFORM-CLOUD-TOKEN"
        }
    }
}
```

We have automated this work around with the following bash script [bin/generate_tfrc_credentials](bin/generate_tfrc_credentials)

## AWS Basics

### S3 Buckets

#### Creating an S3 Bucket

Bucket names must be unique within the global namespace, and can only accept lowercase characters and numbers.