# Restore a Production AMI Backup to a Staging Environment
This assumes we are making nightly AMI backups and a standard DCE single server box.

### 1. Locate the backup
In AWS, it will be in the EC2 console, under `Images` -> `AMIs`. Each AMI should
have a date in its name, e.g., `single_box_prod_on_2018-06-26_07.00.43`

### 2. Launch an instance of the backup
1. Select the AMI and click the "launch" button.
1. Pick a size (t2.large is good for staging).
1. Add it to the DCE Hosting network, and enable `Auto-Assign Public IP`.
1. Leave storage settings the same.
1. Add a tag of `Name` and give it a meaningful value
1. For security groups, `Select an existing security group`. Give it ssh so you can ssh in, and apache so you can connect to the web interface. You might also optionally open up fedora and solr ports here if you are doing some troubleshooting that will require that access. Note, these ports should not be left open for a production environment.
1. Review and launch. You may need to generate a new ssh key if you don't have one of the ones listed. If you are restoring from backup, chances are good everyone's ssh key is already on the box.

### 3. Run instance-specific ansible roles
If you are restoring a backup of the production environment, the box will have production hostname settings, shibboleth certs, etc. Run the last block of roles in the appropriate configuration management ansible playbook. E.g., for Laevigata, that would be these:
  ```
  - { role: set_hostname }
  - { role: emory_dotenv }
  - { role: emory_shibboleth-sp }
  - { role: apache_with_mod_ssl }
  - { role: splunkuforwarder }
  - { role: nrpe, nrpe_version: '3.2.1', nagios_plugins_version: '2.2.1' }
  ```
This will update the hostname, shibboleth cert, ssl cert, splunk configuration, and monitoring configuration to `staging` settings instead of production.

### 4. Reassociate the staging elastic IP
In the AWS EC2 console, go to `Network & Security` --> `Elastic IPs`. Select the elastic IP for the staging environment and re-assign it to the image you just created.

### 5. Check that it worked
You should now be able to interact with your new instance via the staging domain name, but it will have all of the data and machine state of the production environment.

**Note** The process is exactly the same for restoring a production backup to production, except that you don't need to run the ansible roles. 
