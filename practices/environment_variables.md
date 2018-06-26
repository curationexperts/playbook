# Environment Variables

We use environment variables to manage information that varies between different project 
deployments such as hostnames, mail server settings, directory paths, passwords, 
API Keys, and other kinds of data that is either sensitive or instance-specific.  In particular,
sensitive configuration like passwords and API keys should not be checked into 
*publicly accessible* source control (but probably should checked into version control 
somewhere else with appropraite access restrictions).

Here are some good places to read about best practices to which we aspire on this subject:

* [The Rubyist's Guide to Environment Variables](http://blog.honeybadger.io/ruby-guide-environment-variables/)
* [The 12 Factor App](https://12factor.net/)
* [Managing Development Environment Variables Across Multiple Ruby Applications](https://www.twilio.com/blog/2015/02/managing-development-environment-variables-across-multiple-ruby-applications.html)

For concrete details on how DCE handles environment variables in practice, please see the [dotenv setup](https://curationexperts.github.io/playbook/every_project/dotenv.html) section of our playbook.
