# DCE Playbook, a.k.a. "The DCE Way"

## Practices
These are the software development best practices to which we aspire at DCE. This list is a "first day" reading list --
that is, when someone joins the DCE team, these are the practices we want them to get familiar with right away.

- Pair Programming
- Test Driven Development
- [Code Review](first_day/code_review.md)
- [Continuous Integration](first_day/ci.md)
- [CSS & JavaScript Guidelines](practices/css_and_js_guidelines.md)
- [Environment Variables](first_day/environment_variables.md)
- DevOps
- [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)

## Tools

- Code Quality
  - Tools
    - Travis CI
      - [Intermittent Failures](tools/travis/intermittent_failures.md)
    - Gymnasium
    - Coveralls
    - Rubocop
    - Reek(?)

## Every Project
These are things that should be in place for every project
1. [git sha footer](every_project/git_sha.md)
2. [automated deployment](every_project/auto_deploy.md)
3. [exception tracking](every_project/exception_tracking.md)
4. [use dotenv for environment variables](every_project/dotenv.md)
5. [use sidekiq for background jobs](every_project/sidekiq.md)
6. [use okcomputer to make it monitorable](every_project/okcomputer.md)

 - Project Ramp-Up
 - Source Control & Release Process
 - Patterns & Anti-Patterns
 - Trusted Digital Repository Checklist(?)
 - Code Handoff

## Production
Care and feeding of the systems we run
1. [monitoring](production/nagios.md)
