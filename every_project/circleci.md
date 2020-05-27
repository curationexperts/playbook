# CircleCI

## Set up local environment to work with CircleCI
To enable validating `config.yml` syntax before pushing / creating pull request:
1. Just once per dev environment:
  1. Install circleci cli
https://circleci.com/docs/2.0/local-cli/
  1. Create a personal api token on circleci https://app.circleci.com/settings/user/tokens
  1. Run `circleci setup` on the command line, this will create a configuration yml file in your home directory with your circleci host and api token
1. When changing `config.yml` run on command line to ensure correct syntax, etc.
  ```bash
  circleci config validate
  ```

## Set up CircleCI for the application
1. Set up your Dockerhub credentials on the CircleCI application. You can either set these in a context, if you are using multiple related Github projects, or on the project itself.
1. Best practice is to use a machine user, if possible.
Use:
* Environment Variable Name
  * DOCKER_LOGIN
* Value
  * YOUR_INFO_HERE
* Environment Variable Name
  * DOCKER_PASSWORD
* Value
  * YOUR_INFO_HERE

## Set up Github to depend on CircleCI for pushes / merges to master
1. On Github, go to Settings --> Branches, fill in the `Branch name pattern` with `master`, select "Require pull request reviews before merging" and "Require status checks to pass before merging", then select the appropriate ci/circleci tests (as of May 2020 the check that should be selected is `ci/circleci:run-tests`) and save changes.

## Set up repository / application for CircleCI
On the command line, from the root of your application
```bash
mkdir .circleci
touch .circleci/config.yml
```

Example configuration
```yaml
jobs:

  build-and-push:
    executor: docker/docker
    steps:
      - setup_remote_docker
      - checkout
      - docker/check
      - docker/build:
          image: YOUR-DOCKERHUB-ORGANIZATION/YOUR-DOCKERHUB-REPO
      - docker/push:
          image: YOUR-DOCKERHUB-ORGANIZATION/YOUR-DOCKERHUB-REPO
  run-tests:
    docker:
      # Image pulled from registry
      - image: YOUR-DOCKERHUB-ORGANIZATION/YOUR-DOCKERHUB-REPO:$CIRCLE_SHA1
        environment:
            POSTGRES_HOST: localhost
            POSTGRES_DB: YOUR_APPLICATION_DB_TEST
            POSTGRES_USER: postgres
            POSTGRES_PASSWORD: password
            SOLR_CORE: blacklight-test
            SOLR_URL: http://localhost:8983/solr
      - image: YOUR-DOCKERHUB-ORGANIZATION/dc-solr:79a71ec
        command: bash -c 'precreate-core blacklight-test /opt/config; exec solr -f'
      - image: circleci/postgres:9.5-alpine-ram
        environment:
            POSTGRES_DB: YOUR_APPLICATION_DB_TEST
            POSTGRES_USER: postgres
            POSTGRES_PASSWORD: password
    executor: docker/docker
    steps:
      - setup_remote_docker
      - checkout
      - run:
          name: Rubocop
          command: bundle exec rubocop --parallel
      - run:
          name: rspec
          # Excludes tests that require Yale VPN access
          command:  |
            bundle exec rspec
  publish-latest:
    executor: docker/docker
    steps:
      - setup_remote_docker
      - checkout
      - docker/check
      - docker/build:
          image: YOUR-DOCKERHUB-ORGANIZATION/YOUR-DOCKERHUB-REPO
          tag: master
      - docker/push:
          image: YOUR-DOCKERHUB-ORGANIZATION/YOUR-DOCKERHUB-REPO:master
          tag: master
orbs:
  docker: circleci/docker@1.0.1
version: 2.1
workflows:
  commit:
    jobs:
      - build-and-push:
          context: yul-dc
      - run-tests:
          requires:
            - build-and-push
      - publish-latest:
          context: yul-dc
          requires:
            - build-and-push
            - run-tests
          filters:
            branches:
              only: master
```
