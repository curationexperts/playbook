# dotenv setup

We use [dotenv](https://github.com/bkeepers/dotenv) to manage our environment variables, in dev, test, **and production**. In your code, wherever you have a value that shouldn't be checked into version control (e.g., a password), or a value that will vary depending on server name (e.g., a URL config for rails), put it in an environment variable instead and reference it by calling `ENV['WHATEVER']` in your code.

1. Add `dotenv-rails` to your gemfile at the top level. Do not restrict it to only being in the dev and test groups. Run `bundle install`
2. Create a file called `dotenv.sample` in the root of your project. You'll populate this with empty values for every environment variable you end up setting, so a new user of the project has a starting place.
3. Create a file called `.env.development` to hold your local development settings
4. Create a file called `.env.production` to hold the settings for the production server(s)
5. Add `.env.production` and `.env.production` to `.gitignore`
6. Whenever you need to reference a value, add it to those files and reference it as an environment variable
7. Add .env.production to your Capistrano deploy's set of linked files. You'll need to copy .env.production to your server's shared directory (where the linked files live) manually.
