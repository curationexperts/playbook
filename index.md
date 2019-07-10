# DCE Playbook, a.k.a. "The DCE Way"

## Practices
These are the software development best practices to which we aspire at DCE. This list is a "first day" reading list --
that is, when someone joins the DCE team, these are the practices we want them to get familiar with right away.

- [Code Review](practices/code_review.md)
- [Continuous Integration](practices/ci.md)
- [CSS & JavaScript Guidelines](practices/css_and_js_guidelines.md)
- [Environment Variables](practices/environment_variables.md)
- [Estimation](practices/estimation.md)
- [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)
- [Team Definition of Done](practices/done.md)

## Tools

- Travis CI
  - [Intermittent Failures](tools/travis/intermittent_failures.md)

## Every Project
These are things that should be in place for every project
1. [project management with waffle](every_project/waffle.md)
1. [git sha footer](every_project/git_sha.md)
1. [automated deployment](every_project/auto_deploy.md)
1. [exception tracking](every_project/exception_tracking.md)
1. [use dotenv for environment variables](every_project/dotenv.md)
1. [use sidekiq for background jobs](every_project/sidekiq.md)
1. [sample data](every_project/sample_data.md)
1. [clean up temp files](every_project/cleanup_temp_files.md)
1. [use okcomputer to make it monitorable](every_project/okcomputer.md)
1. [geonames web service](every_project/geonames.md)

## Production
Care and feeding of the systems we run
1. [monitoring](production/nagios.md)
2. [How to clean out Hyrax](practices/cleanout_hyrax.md)
3. [How to move data from one Hyrax to another](production/backup_and_restore.md)
4. [Sidekiq in production](production/sidekiq_in_production.md)
5. [Defining a new service in nagios](production/define_new_service.md)
6. [Restarting services automatically with nagios](production/restart_services.md)
7. [AWS Nightly Backups](production/ami_backups.md)
8. [Restore Production Backup to Staging](production/restore_prod_to_stage.md)
9. [Continuous Deploy with Travis](production/continuous_deployment.md)
10. [Email on AWS](production/aws_email.md)
11. [Exception Logging](production/exception_logging.md)

## Authentication
1. [Shibbboleth](authentication/shibboleth.md)
1. [LDAP](authentication/ldap.md)
