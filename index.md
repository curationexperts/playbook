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

## Tools

- Travis CI
  - [Intermittent Failures](tools/travis/intermittent_failures.md)

## Every Project
These are things that should be in place for every project
1. [git sha footer](every_project/git_sha.md)
2. [automated deployment](every_project/auto_deploy.md)
3. [exception tracking](every_project/exception_tracking.md)
4. [use dotenv for environment variables](every_project/dotenv.md)
5. [use sidekiq for background jobs](every_project/sidekiq.md)
6. [clean up temp files](every_project/cleanup_temp_files.md)
7. [use okcomputer to make it monitorable](every_project/okcomputer.md)
8. [geonames web service](every_project/geonames.md)

## Production
Care and feeding of the systems we run
1. [monitoring](production/nagios.md)
2. [How to clean out Hyrax](practices/cleanout_hyrax.md)
3. [Sidekiq in production](production/sidekiq_in_production.md)

## Authentication
1. [Shibbboleth](authentication/shibboleth.md)
