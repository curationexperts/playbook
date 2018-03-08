# Shibboleth IDP
DCE maintains a shibboleth idp server so that we can configure applications against 
shibboleth without having to rely on client infrastructure. Shibboleth Identity Provider (a.k.a. "idp") runs as a Tomcat8 servlet on meredith.curationexperts.com.

Our shibboleth end point is at [https://meredith.curationexperts.com/idp/shibboleth](https://meredith.curationexperts.com/idp/shibboleth).  

## How to build the shibboleth server
The shibboleth server is built via an ansible playbook at https://github.com/curationexperts/dce-cm/ (private). 

Most of the server build is automated, but there are some steps that must be
done manually:

1. Set the ldap admin password
  * Eventually we'd like to automate this. We are tracking the issue [here](https://github.com/curationexperts/dce-cm/issues/1). Run the ansible build. It will fail when it tries to 
  create the first ldap user, with the error `ldap_bind: Invalid credentials`
  * To set the ldap admin password manually:
  ```
  ssh ubuntu@meredith.curationexperts.com (or whatever hostname you're using)
  sudo dpkg-reconfigure slapd
  ```
  Most of the settings will already have been configured by ansible. The `dpkg-reconfigure` command will take you screen by screen through the settings. Accept the setting for every screen except the password screens, where you should paste the password that ansible is expecting to use. This should be set in the ansible `hosts` file, and you'll also see it in the error message ansible gave you.
  
    You can then re-run the ansible build, starting at the beginning of the LDAP module, like this:
  ```
  ansible-playbook build_dce_ldap.yml --start-at-task="ldap : install slapd"
  ```

### Notes
* To see what attributes shibboleth is releasing, visit this url: 
https://IDP_HOSTNAME/idp/profile/admin/resolvertest?requester=blah&principal=mark 
