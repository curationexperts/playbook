# Automatically deploy using Capistrano

Code deploys should be frequent and automated. Where ever possible, this means using capistrano.

It's likely you'll be deploying to a server that was built with `ansible-samvera` build scripts, in which case, please follow this [Capifying](https://curationexperts.github.io/ansible-samvera/capification.html) guide.

## Deploy Targets
For DCE projects, the standard deploy targets are:
* **cd** -  Stands for "Continuous Deploy". The current version of the code is automatically deployed on a cron job (usually every 24 hours). Not used for all projects.
* **qa** - Stands for "Quality Assurance". Used for acceptance testing; the client (sometimes an internal DCE client) can sign off that what they're seeing is good to move forward, or give feedback for further work needed.
* **stage** - Short for "Staging". Pre-production checking for successful deployment. Should be as close as possible to production environment.
* **prod** - Short for "Production". Where the application is accessed by end users. Should be as stable and seamless for users as possible.

*Note: This documents practice going forward. There are some older systems that currently use 'dev' as a stage name, e.g. tenejo-dev.curationexperts.com. We have moved away from using "dev" as a stage because it can cause confusion with the phrase "development environment" (which is the developer's local computer, **not** a server or deploy target).*
