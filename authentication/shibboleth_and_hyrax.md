# Authenticating with Shibboleth in a Hyrax Application

*Goal:* Authenticate against shibboleth in production, but against a database in (local) development environment. 

### 1. Add needed fields to your user model
* Add to `spec/models/user_spec.rb:`
```ruby
describe 'omniauthable user' do
  it "has a uid field" do
    expect(user.uid).not_to be_empty
  end
  it "can have a provider" do
    expect(described_class.new.respond_to?(:provider)).to eq true
  end
end
```
* Run on command line:

  ```bash
  $ rails g migration AddOmniauthToUsers provider:string uid:string:index
  $ rake db:migrate
  ```
* Add to `spec/factories/users.rb`:
```ruby
sequence :uid do |n|
  "#{FFaker::Internet.user_name}#{n}"
end
```
* Now your new test should pass

### 2. Use uid instead of email for the user_key
Edit `config/initializers/devise.rb` and change the value of `config.authentication_keys` to `uid` (or whatever is appropriate for this particular shibboleth integration.)
```
config.authentication_keys = [:uid]
```
Run your test suite and fix any tests that broke. You might need to use `find_by_user_key` instead of find_by(:email), for example.

### 3. Add AuthConfig model
Add an model called AuthConfig we can use to configure which authentication method we want to use. This lets us continue to use database authentication in development. Add this to `app/models/auth_config.rb`:
  ```ruby
  class AuthConfig
    # In production, we use Shibboleth for user authentication,
    # but in development mode, you may want to use local database
    # authentication instead.
    def self.use_database_auth?
      !Rails.env.production? && ENV['DATABASE_AUTH'] == 'true'
    end
  end
  ```

### 4. Allow shibboleth login to create a new user
We can't assume that all user accounts will exist on the system before they log in, so authenticating against shibboleth has to allow for the creation of a new User account at login time. 

* Add your tests first, to `/spec/models/user_spec.rb`

  ```ruby
  context "shibboleth integration" do
    let(:auth_hash) do
      OmniAuth::AuthHash.new(
        provider: 'shibboleth',
        uid: "janeq",
        info: {
          display_name: "Jane Quest",
          uid: 'janeq',
          mail: 'janeq@example.com'
        }
      )
    end
    let(:user) { described_class.from_omniauth(auth_hash) }

    before do
      described_class.delete_all
    end

    context "shibboleth" do
      it "has a shibboleth provided name" do
        expect(user.display_name).to eq auth_hash.info.display_name
      end
      it "has a shibboleth provided uid which is not nil" do
        expect(user.uid).to eq auth_hash.info.uid
        expect(user.uid).not_to eq nil
      end
      it "has a shibboleth provided email which is not nil" do
        expect(user.email).to eq auth_hash.info.mail
        expect(user.email).not_to eq nil
      end
    end
  end
  ```
  * add the `from_omniauth` method to `app/models/user.rb`:
    ```ruby
    # When a user authenticates via shibboleth, find their User object or make
    # a new one. Populate it with data we get from shibboleth.
    # @param [OmniAuth::AuthHash] auth
    def self.from_omniauth(auth)
      Rails.logger.debug "auth = #{auth.inspect}"
      # Uncomment the debugger above to capture what a shib auth object looks like for testing
      user = where(provider: auth.provider, uid: auth.info.uid).first_or_create
      user.display_name = auth.info.display_name
      user.uid = auth.info.uid
      user.email = auth.info.mail
      user.save
      user
    end
    ```
* Now your `spec/models/user_spec.rb` test should pass.

### 5. Devise integration

* Add `omniauth-shibboleth` to your `Gemfile` and run `bundle install`:
```
gem 'omniauth-shibboleth', '~> 1.3'
```

* replace the devise modules already in `app/models/user.rb` with the ones below. 
  ```ruby
  # Include devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  # remove :database_authenticatable in production, remove :validatable to integrate with Shibboleth
  devise_modules = [:omniauthable, :rememberable, :trackable, omniauth_providers: [:shibboleth], authentication_keys: [:uid]]
  devise_modules.prepend(:database_authenticatable) if AuthConfig.use_database_auth?
  devise(*devise_modules)
  ```

  * Run your tests. You will have tests failing with a message like `undefined method destroy_user_session_path`. That's because you haven't added omniauth routes yet. 
  
### 5. Add shibboleth routes, controllers, and devise configuration

* Edit `config/routes.rb` and replace the line `devise_for :users` with:
```ruby

  devise_for :users, controllers: { omniauth_callbacks: "omniauth_callbacks" }

  # Disable these routes if you are using Devise's
  # database_authenticatable in your development environment.
  unless AuthConfig.use_database_auth?
    devise_scope :user do
      get 'sign_in', to: 'omniauth#new', as: :new_user_session
      post 'sign_in', to: 'omniauth_callbacks#shibboleth', as: :new_session
      get 'sign_out', to: 'devise/sessions#destroy', as: :destroy_user_session
    end
  end
```
* Now create some controllers to respond to these routes:
* Create `app/controllers/omniauth_controller.rb`:
```ruby
  class OmniauthController < Devise::SessionsController
    def new
      # Rails.logger.debug "SessionsController#new: request.referer = #{request.referer}"
      if Rails.env.production?
        redirect_to user_shibboleth_omniauth_authorize_path
      else
        super
      end
    end
  end
```
* Create `app/controllers/omniauth_callbacks_controller.rb`:
```ruby
  class OmniauthCallbacksController < Devise::OmniauthCallbacksController
    def shibboleth
      Rails.logger.debug "OmniauthCallbacksController#shibboleth: request.env['omniauth.auth']: #{request.env['omniauth.auth']}"
      # had to create the `from_omniauth(auth_hash)` class method on our User model
      @user = User.from_omniauth(request.env["omniauth.auth"])
      set_flash_message :notice, :success, kind: "Shibboleth"
      sign_in_and_redirect @user
    end
  end
```
* Configure devise / omniauth to use shibboleth. Add these lines to the OmniAuth configuration section of `config/initializers/devise.rb`:
```ruby
config.omniauth :shibboleth,
                uid_field: 'uid',
                info_fields: { display_name: 'displayName', uid: 'uid', mail: 'mail' },
                callback_url: '/users/auth/shibboleth/callback',
                strategy_class: OmniAuth::Strategies::Shibboleth
```
* Your test suite should now pass.

### 6. Allow user login with uid
* Add to `app/views/devise/sessions/new.html.erb`:
```ruby
  <h2>Log in</h2>

  <%= render "devise/shared/links" %>

  <% if AuthConfig.use_database_auth? %>
    <%= simple_form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>
      <div class="form-inputs">
        <%= f.input :uid, required: true, autofocus: true %>
        <%= f.input :password, required: true %>
        <%= f.input :remember_me, as: :boolean if devise_mapping.rememberable? %>
      </div>

      <div class="form-actions">
        <%= f.button :submit, "Log in" %>
      </div>
    <% end %>
  <% end %>
```

### 7. Allow systems users to be created without passwords
Now in production we're expecting that all users will be managed with shibboleth, 
and so the `User` model no longer has a `password` method. This is going to cause
anything that creates a systems user to fail. Let's fix that.

* Write the test first, in `spec/models/user_spec.rb`:
```ruby
context "in a world without passwords" do
  before do
    described_class.delete_all
  end
  it "system users are created without error" do
    allow(AuthConfig).to receive(:use_database_auth?).and_return(false)
    u = ::User.find_or_create_system_user("batch_user")
    expect(u).to be_instance_of(::User)
  end
end
```

* Then add this method to the very end of `app/models/user.rb`, *outside* the class
block:
```ruby
# Override a Hyrax class that expects to create system users with passwords.
# Since in production we're using shibboleth, and this removes the password
# methods from the User model, we need to override it.
module Hyrax::User
  module ClassMethods
    def find_or_create_system_user(user_key)
      u = ::User.find_or_create_by(uid: user_key)
      u.display_name = user_key
      u.email = "#{user_key}@example.com"
      u.password = ('a'..'z').to_a.shuffle(random: Random.new).join if AuthConfig.use_database_auth?
      u.save
      u
    end
  end
end
``` 

* Edit `config/initializers/hyrax.rb` and change the user_key for the batch user and the audit user to something that isn't an email address (e.g., "batchuser" and "audituser").
  
You should now be able to deploy this application to a systems with Shibboleth SP configured and have it work as expected. Note that this document assumes the systems to which you'll be deploying is set up in the DCE Shibboleth SP pattern. 
