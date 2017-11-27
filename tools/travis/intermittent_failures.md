# Dealing with intermittent failures on Travis CI

DCE uses Travis CI to run specs when pushing to GitHub. Travis
is a unique environment that is slightly different than our 
production environments or development machines. You may encounter
intermittent failures when specs run on Travis that you don't run into on your 
local machine. This problem is pronounced when running feature tests that 
rely on JavaScript. 

If you have a spec that is passing locally, but sometimes failing on Travis you can use a strategy that Chris Colvard came up with on the Avalon project. The original Avalon commit for this functionality is [here](https://github.com/avalonmediasystem/avalon/commit/2a5b21dcb7d40c47a77acf6fb24de089944588b5).

You add this module to `spec/support`:

```ruby
module OptionalExample
  RSpec.configure do |config|
    config.after do |example|
      if example.metadata[:optional] && (RSpec::Core::Pending::PendingExampleFixedError === example.display_exception)
        ex = example.display_exception
        example.display_exception = nil
        example.execution_result.pending_exception = ex
      end
    end
  end

  def optional(message)
    RSpec.current_example.metadata[:optional] = true
    pending(message)
  end
end
```

Then include that module in the `rails_helper`: 

```ruby 
config.include OptionalExample
```

This allows you to make a test as optional for a specific enviroment. It is similar to pending, but will not fail if the test passes. Here's how to use it in a spec:

```ruby
it 'should delete existing tokens' do
 optional "Sometimes fails on travis" if ENV['TRAVIS']
 expect(false).to be_falsey
end
```
