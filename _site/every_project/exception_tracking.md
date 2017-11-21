# Exception Tracking

In addition to having automated tests for our software, we also want to track any exceptions that are being generated by the system in production. There will always be situations we didn't anticipate, and learning about errors as soon as possible, as well as knowing how often they are occurring and under what circumstances, lets us be responsive to the real world circumstances in which our software runs.

Our current chosen tool for exception tracking is [Honeybadger](https://app.honeybadger.io).

Start by reading the [Honeybadger for Ruby](http://docs.honeybadger.io/ruby/index.html) documentation for an overview of how Honeybadger works.

To add a project to honeybadger:

1. Get someone to add you to the honeybadger team. (See https://app.honeybadger.io/teams/34396)
2. Add a project at https://app.honeybadger.io/projects. **If this is a hosted project create a separate project for the production instance**. Give the project a meaningful name. Say it's a Ruby project (assuming it is) and retain logs for 30 days.
3. Follow the `Rails Installation` instructions that will come up. Run the test and ensure you can send a test exception.
4. Do NOT check the `config/honeybadger.yml` file that was created into version control. Instead, add the API key that's specific to this project to the `/etc/environment` file on all servers where this project will be deployed. Eventually this should be managed by ansible, but for now you can do it by hand. Add this line to the end of `/etc/environment`, replacing the API key with the one for your project:

  ```
    HONEYBADGER_API_KEY=mst3k
  ```
5. To ensure you can send exceptions from a given server, log into the system as the deploy user and send a test exception:

  ```
    $ RAILS_ENV=production bundle exec honeybadger test
    Raising 'HoneybadgerTestingException' to simulate application failure.
    ⚡ Success: https://app.honeybadger.io/notice/2d77f413-f4de-4f8b-b3f2-258fc69c0e3f
  ```
6. You'll need to restart apache for the environment variable to be available to the running service. If this is a production server, you can schedule the restart to happen later, using the `at` command. Make sure you include the full path to the `service` command:

  ```bash
    root@qa-etd:/opt/laevigata$ echo "/usr/sbin/service apache2 restart" | at 0500 nov 16
    warning: commands will be executed using /bin/sh
    job 3 at Thu Nov 16 10:15:00 2017
    root@qa-etd:/opt/laevigata$ atq
    3	Thu Nov 16 05:00:00 2017 a root
  ```