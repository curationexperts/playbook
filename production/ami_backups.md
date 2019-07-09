# Setting up AMI Backups on AWS
In production, we prefer to make AMI images for our nightly backups. This lets
us recover quickly in case of emergency, and it preserves not only the data but
also the machine state.

We use AWS `Systems Manager` to automate the creation of AMIs.

Here's how to set it up:

### 1. Create a Systems Manager AIM role
Amazon maintains documentation for this process at https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-maintenance-perm-console.html

Create an EC2 role called `AWSSystemsManager`.

This role needs to have enough permissions to create a maintenance window, to restart the box etc. These are the correct set of permissions:

![Role Permissions](/assets/role-permissions.png)

AmazonEC2FullAccess
AmazonEC2RoleForSSM
AmazonSSMFullAccess
AmazonSSMMaintenanceWindowRole

Make sure that the trust relationship is set:

Choose the Trust relationships tab, and then choose Edit trust relationship.

Delete the current policy, and then copy and paste the following policy into the Policy Document field:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "ec2.amazonaws.com",
          "ssm.amazonaws.com",
          "sns.amazonaws.com"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Choose Update Trust Policy, and then copy or make a note of the role name and the Role ARN value on the Summary page. You will specify this information when you create your Maintenance Window.

### 2. Add that role to each instance you want a nightly backup for
In the EC2 console, select the box and use the `Attach/Replace AIM Role` menu option.

### 3. Ensure AWS SSM

As of early 2019, AWS includes ssm by default in the Ubuntu 16.04 and 18.04 AMIs.

You can verify that it is running with this command:

`sudo systemctl status snap.amazon-ssm-agent.amazon-ssm-agent.service`

After adding the role to the instance, restart the ssm agent:

`sudo systemctl restart snap.amazon-ssm-agent.amazon-ssm-agent.service`

If you aren't using a newer Ubuntu 16.04 or 18.04 image you can use an ansible role similar to this:
https://github.com/curationexperts/emory-cm/blob/master/roles/aws_ssm/tasks/main.yml


### 4. Create (or check) Backup document
In AWS, go to `Systems Manager` and select `Document` from the menu on the left.
Check to see if you have the Document you need by searching for "Owned by me". If
you don't have a document called `single_box_backup`, make an *automation* document with [this content](/aws-systems-manager-backup.json).

If you need to edit this, create a new verison and assign the new version as the default.

### 5. Create a Maintenance Window that uses that document
1. Create a maintenance window with an appropriate name
2. Assign it an "Automation Task" that references the `single_box_backup` document you made in the above step
3. Give it the instance id of the EC2 instance you want to back up, along with a name and description for the AMIs
4. Give it a schedule. Our nagios monitoring is configured to ignore outages from 3 - 4am Eastern Time. The cron expression to match that is `cron(0 0 7 ? * * *)`.
