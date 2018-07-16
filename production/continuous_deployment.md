# Continuous Deployment with Travis and Capistrano
For many of our project it will be desirable to have the latest master code deployed to a server for QA review without the need for manual intervention.
Here's how to set that up.

## 1. Ensure you can deploy to the server in question.
In this case we'll assume a server called `cd.curationexperts.com` and a cap
deploy target of `cd`. That implies that one should be able to deploy code via:
```
bundle exec cap cd deploy
```
We'll use the `laevigata` github repo as an example.

## 2. Ensure you have a working travis CI build.

## 3. Generate an rsa key pair to use for deployment
```
$ ssh-keygen -t rsa -b 4096 -C "administrator@curationexperts.com"
 Enter file in which to save the key (/home/bess/.ssh/id_rsa): ~/.ssh/laevigata_deploy_rsa
 Enter passphrase (empty for no passphrase): (leave empty)
 Enter same passphrase again: (leave empty)
```

## 4. Add the public key to the deploy user on the cd server
Append the output of `cat ~/.ssh/laevigata_deploy_rsa.pub` to `/home/deploy/.ssh/authorized_keys` on the server to which you want
travis to deploy.

## 6. Encrypt the deploy key you just created
Use travis provided tools to encrypt the private key.
1. Install travis tools locally: `sudo apt-get install travis`
1. Run `travis login` and authenticate to github.
1. `travis encrypt-file YOUR_PRIVATE_KEY --add --repo=curationexperts/laevigata`
1. This will make changes to your `.travis.yml` file, including a command to de-crypt the file. Make sure these get saved and committed.
1. Make sure you add the encrypted version of your key to version control, and NOT the original

## 7. Tell travis to deploy after a successful build of master
Add a block to `.travis.yml` that will run a capistrano deploy:
```
after_success:
  - |
    if [[ $TRAVIS_BRANCH == 'master' && $TRAVIS_PULL_REQUEST == 'false' ]]; then
      bundle exec cap prod_upgrade deploy
    fi
```

## 8. Tell capistrano to use the rsa key
Add this line to `config/deploy.rb`:
```
set :ssh_options, keys: ["deploy_id_rsa"] if File.exist?("deploy_id_rsa")
```

## 9. Commit your changes
You should have a changed `.travis.yml` and `config/deploy.rb`. You should also have a new, encrypted, rsa key. Make a PR with these changes and when it is merged to master it should automatically deploy.
