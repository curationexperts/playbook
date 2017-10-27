### Capify a Rails app the DCE Way

1. Check out the project and ensure the tests pass
2. On a branch, go through the 'quick start' capistrano installation guide, here: https://github.com/capistrano/capistrano#quick-start
  * Make some DCE specific stages, instead of just the defaults: `bundle exec cap install STAGES=localhost,sandbox,qa,staging,production`
