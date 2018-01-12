# Shibboleth SP
The server that will be authenticating against a Shibboleth IDP system needs to 
be configured as a Shibboleth SP (service provider). We have an ansible role 
that sets this up at https://github.com/curationexperts/dce-cm/tree/master/roles/shibboleth-sp (private).

Mainly this involves installing `libapache2-mod-shib2` and its pre-requisites 
and configuring it against the Shibboleth IDP server you want. 

## Testing
For best results, ensure you can authenticate against Shibboleth at https://YOUR_SERVER_NAME/Shibboleth.sso/Login 
before you try to integrate Shibboleth authn into Hyrax.
