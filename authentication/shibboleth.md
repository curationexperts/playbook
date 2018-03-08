# Shibboleth

In order to use Shibboleth authentication for a Samvera app, three things must
be in place:

1. A shibboleth-idp (identity provider) server. We maintain one for internal use. See 
the [shibboleth-idp](shibboleth-idp.md) guide for details.
2. A shibboleth-sp (service provider) installation on the server where the application
will run. See the [shibboleth-sp](shibboleth-sp.md) guide for details.
3. The application must be configured to authenticate with shibboleth.  See the 
[shibboleth_and_hyrax](shibboleth_and_hyrax.md) guide for how to do this.
