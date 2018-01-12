# Geonames

Hyrax Basic Metadata uses the Geonames web service as the authority for the based_near property ("Location"). We regression test with Basic Metadata, and additionally maintain the nurax.curationexperts.com site which demonstrates how Out-of-the-Box Hyrax installations look and function.

## Setting up geonames locally

1. Add a `GEONAMES_USER` environment variable for the dce_tester account in your project's .env.development and .env.test files.

2. Add that to the Hyrax.rb initializer: `config.geonames_username = ENV['GEONAMES_USER']`

3. Restart your web server, login as an admin and click new work, and you should be able to see autocomplete results in the Location select. Or run the test suite with a test that confirms the Location (based_near) Metadata property.

4. See [Hyrax Geonames Setup] (https://github.com/samvera/hyrax/wiki/Hyrax-Management-Guide#geonames) for more information.
