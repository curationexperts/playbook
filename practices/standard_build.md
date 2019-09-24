# DCE Standard Build

We strive for consistency, reproducibility, and maintainability in our system builds. We are heavily influcenced by books like [The DevOps Handbook](https://www.oreilly.com/library/view/the-devops-handbook/9781457191381/) and [The Phoenix Project](https://www.goodreads.com/book/show/17255186-the-phoenix-project).

When building systems, we strive to build everything via ansible scripts. Our goal is to never build or even tweak settings by hand, as this leads to bespoke and unpredictable systems.  

The core of our build is in [ansible-samvera](https://github.com/curationexperts/ansible-samvera). DCE-specific configuration (e.g., DCE SSL certs) is in a private repo called [dce-cm](https://github.com/curationexperts/dce-cm). Client specific server builds will be in private repos named appropriately, which pull from ansible-samvera and dce-cm as needed.

## Standard build profile

### After September 2019 - ansible samvera > 1.5.0

|Software|version|Notes  |   |   |
|----|---|---|---|---|
|**Operating System**|
| Ubuntu  |18.04 LTS   |   |   |   |
|**Ruby**|   |  |   |   |
|Ruby| 2.6.3 |   |   |   |
|Passenger| 6.0.4 |   |   |   |
|**Database**|   |   |   |   |
|Postgresql| latest  |   |   |   |
|redis| latest  |   |   |   |
|**Fedora Repository**|   |We back fedora with Postgresql   |   |   |
|Tomcat| 8.x  |   |   |   |
|Fedora| 4.7.5  |   |   |   |
|**Solr**|   |We run solr as a standalone service, not inside of Tomcat   |   |   |
|Solr| 7.7.1 |   |    |   |
|**Graphics**|  | We compile ImageMagick with these specific library versions because through trial and error we have determined this produces the widest range of acceptable derivative images   |   |   |
|ImageMagick| 7.0.7  |   |   |   |
|gs (ghostscript)| 9.19  |   |   |   |
|openjpeg| 2.1.0  |   |   |   |
|libtiff| 4.0.5  |   |   |   |
|libpng| 1.6.34 |   |   |   |
|**FITS**|   |We install FITS as both a standalone command-line script, and as a Tomcat 8 servlet. |   |   |
|FITS| 1.4.0 |   |   |   |


### Before September 2019 - ansible samvera <= 1.5.0

|Software|version|Notes  |   |   |
|----|---|---|---|---|
|**Operating System**|
| Ubuntu  |16.04 LTS   |   |   |   |
|**Ruby**|   |  |   |   |
|Ruby| 2.4.2 |   |   |   |
|Passenger| 5.3.2 |   |   |   |
|**Database**|   |   |   |   |
|Postgresql| latest  |   |   |   |
|redis| latest  |   |   |   |
|**Fedora Repository**|   |We back fedora with Postgresql   |   |   |
|Tomcat| 7.x  |   |   |   |
|Fedora| 4.7.5  |   |   |   |
|**Solr**|   |We run solr as a standalone service, not inside of Tomcat   |   |   |
|Solr| 6.6.2 |   |    |   |
|**Graphics**|  | We compile ImageMagick with these specific library versions because through trial and error we have determined this produces the widest range of acceptable derivative images   |   |   |
|ImageMagick| 7.0.7  |   |   |   |
|gs (ghostscript)| 9.19  |   |   |   |
|openjpeg| 2.1.0  |   |   |   |
|libtiff| 4.0.5  |   |   |   |
|libpng| 1.6.34 |   |   |   |
|**FITS**|   |  |   |   |
|FITS| 0.8.4 |   |   |   |
