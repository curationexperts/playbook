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
you don't have a document called `single_box_backup`, make one with [this content](/aws-systems-manager-backup.json).

### 5. Create a Maintenance Window that uses that document
1. Create a maintenance window with an appropriate name
2. Assign it an "Automation Task" that references the `single_box_backup` document you made in the above step
3. Give it the instance id of the EC2 instance you want to back up, along with a name and description for the AMIs
4. Give it a schedule. Our nagios monitoring is configured to ignore outages from 3 - 4am Eastern Time. The cron expression to match that is `cron(0 0 7 ? * * *)`.
