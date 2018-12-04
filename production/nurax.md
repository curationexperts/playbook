# Nurax (nurax-dev and nurax-stable)
DCE maintains the "nurax" servers for the Samvera community for the purposes of
testing Hyrax development and releases.

## AWS Location
The instances are located in the DCE AWS main space, in the Ohio region.

## Build and AMIs
Nurax instances can be built from scratch with the ansible scripts in `dce-cm`.
These are private because they contain certificates and passwords, but they are
based largely on the scripts at `ansible-samvera`, which are public.

E.g.:
```
ansible-playbook nurax.yml --extra-vars "host=nurax-dev"
```


The easiest way to get an additional nurax server, however, is probably to shut
down an existing one and create an AMI snapshot of it, then use that AMI snapshot
to launch a new instance. See the instructions in `restore_prod_to_stage.md` for
more details.

## Continuous deployment
