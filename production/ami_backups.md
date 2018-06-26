# Setting up AMI Backups on AWS
In production, we prefer to make AMI images for our nightly backups. This lets
us recover quickly in case of emergency, and it preserves not only the data but
also the machine state.

We use AWS `Systems Manager` to automate the creation of AMIs.

Here's how to set it up:

### 1. Create a Systems Manager AIM role
Amazon maintains documentation for this process at https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-maintenance-perm-console.html

### 2. Add that role to each instance you want a nightly backup for
In the EC2 console, select the box and use the `Attach/Replace AIM Role` menu option.

### 3. Ensure AWS SSM
This should be part of our standard build, but in case you don't have it installed,
you can follow these directions:
https://github.com/curationexperts/emory-cm/blob/master/roles/aws_ssm/tasks/main.yml

### 4. Create (or check) Backup document
In AWS, go to `Systems Manager` and select `Document` from the menu on the left.
Check to see if you have the Document you need by searching for "Owned by me". If
you don't have a document called `single_box_backup`, make one with this content:
```json
{
  "schemaVersion": "0.3",
  "description": "Creates AMI snapshot of a given instance id",
  "parameters": {
    "InstanceIamRole": {
      "type": "String",
      "description": "(Required) The name of the role that enables Systems Manager (SSM) to manage the instance.",
      "default": "AWSSystemsManager"
    },
    "AutomationAssumeRole": {
      "type": "String",
      "description": "(Required) The ARN of the role that allows Automation to perform the actions on your behalf.",
      "default": "arn:aws:iam::484534655265:role/AWSSystemsManager"
    },
    "InstanceId": {
      "type": "String",
      "description": "(Required) The EC2 instance ID."
    },
    "AmiName": {
      "type": "String",
      "description": "(Required) The name of the AMI that will be created"
    },
    "AmiDescription": {
      "type": "String",
      "description": "(Required) Description for the AMI that will be created"
    }
  },
  "mainSteps": [
    {
      "name": "stopInstances",
      "action": "aws:changeInstanceState",
      "maxAttempts": 3,
      "timeoutSeconds": 1200,
      "onFailure": "Abort",
      "inputs": {
        "InstanceIds": [
          "{{InstanceId}}"
        ],
        "DesiredState": "stopped"
      }
    },
    {
      "name": "createImage",
      "action": "aws:createImage",
      "maxAttempts": 3,
      "onFailure": "Abort",
      "inputs": {
        "InstanceId": "{{InstanceId}}",
        "ImageName": "{{AmiName}}_on_{{global:DATE_TIME}}",
        "NoReboot": true,
        "ImageDescription": "{{AmiDescription}} on {{global:DATE_TIME}} from {{InstanceId}}"
      }
    },
    {
      "name": "startInstances",
      "action": "aws:changeInstanceState",
      "maxAttempts": 3,
      "timeoutSeconds": 1200,
      "onFailure": "Abort",
      "inputs": {
        "InstanceIds": [
          "{{InstanceId}}"
        ],
        "DesiredState": "running"
      }
    }
  ],
  "outputs": [
    "createImage.ImageId"
  ]
}
```

### 5. Create a Maintenance Window that uses that document
1. Create a maintenance window with an appropriate name
2. Assign it an "Automation Task" that references the `single_box_backup` document you made in the above step
3. Give it the instance id of the EC2 instance you want to back up, along with a name and description for the AMIs
4. Give it a schedule. Our nagios monitoring is configured to ignore outages from 3 - 4am Eastern Time. The cron expression to match that is `cron(0 0 7 ? * * *)`.
