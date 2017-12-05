# okcomputer

We use [okcomputer](https://github.com/sportngin/okcomputer) to expose a health check endpoint on our applications. Doing this allows us to more easily set up [nagios monitoring](../production/nagios.md) for the application itself and also for any services it requires for proper functioning. For example, if your application sends mail via an external SMTP server, it's a good idea to monitor whether it can connect to that server and get a response as expected, without actually having to send and check email.

## installation

Install okcomputer according to [the instructions in the README](https://github.com/sportngin/okcomputer). Basically, you just add `gem okcomputer` to your gemfile and run `bundle install`. Then, configuration goes in `config/initializers/okcomputer.rb`

## using built-in checks

Okcomputer comes with many pre-configured checks you can load. See [the full list on github](https://github.com/sportngin/okcomputer/tree/master/lib/ok_computer/built_in_checks). Each includes relevant configuration options. For example, here's how to check whether the handle sidekiq queue is backing up, i.e., is there more than 360 seconds of latency between the time a job enters the queue and the time it leaves. In other words, signal a problem if the oldest job in the queue has been waiting longer than 6 minutes.

```ruby
  # config/initializers/okcomputer.rb
  OkComputer::Registry.register "handle_sidekiq_queue_latency", OkComputer::SidekiqLatencyCheck.new('handle', 360)
```

## writing checks

Here is an example SMTP check. Notice that it gets its values from dotenv, giving us an easy way to check that dotenv is set up correctly on any server where this is expected to work. Also notice that we rescue exceptions, report the exception to honeybadger, and mark the check as a failure if an exception is raised. This check will create an endpoint at `/okcomputer/smtp`.

```ruby
  # config/initializers/okcomputer.rb
  require 'honeybadger'

  class SmtpCheck < OkComputer::Check
    def check
      smtp = Net::SMTP.new ENV['ACTION_MAILER_SMTP_ADDRESS'], ENV['ACTION_MAILER_PORT']
      smtp.enable_starttls
      smtp.start('aws.emory.edu', ENV['ACTION_MAILER_USER_NAME'], ENV['ACTION_MAILER_PASSWORD'], :plain) do |s|
        mark_message "SMTP connection working"
      end
    rescue => exception
      Honeybadger.notify(exception)
      mark_failure
      mark_message "Cannot connect to SMTP"
    end
  end

  OkComputer::Registry.register "smtp", SmtpCheck.new
```
