# Setting up LDAP Authentication
The DCE way of setting up LDAP for client demo purposes. We set up
[ladle](https://github.com/NUBIC/ladle) to run a local LDAP for testing against.

### 1. Install hydra-role-management
Install the [hydra-role-management](https://github.com/samvera/hydra-role-management) gem according to the instructions on that gems's website.

### 2. Ensure you have a user test suite in place.
1. Create a shared example for what a hyrax user looks like in `spec/support/shared_examples/hyrax_user.rb`:
  ```ruby
  RSpec.shared_examples 'a Hyrax User' do
    describe '#ability' do
      it 'has abilities managed by Ability' do
        expect(user.ability).to be_a Ability
      end
    end

    describe '#email' do
      it 'is the set email' do
        expect { user.email = 'example.user@test.com' }
          .to change { user.email }
          .to 'example.user@test.com'
      end
    end

    describe '#password' do
      it 'is the set password' do
        expect { user.password = 'secret' }
          .to change { user.password }
          .to 'secret'
      end
    end
  end
  ```
1. Create a test suite for your `User` model in `spec/models/user_spec.rb` and ensure it runs:
  ```ruby
  require 'rails_helper'

  RSpec.describe User do
    subject(:user) { build(:user) }
    let(:plain_user) { create(:user) }

    it_behaves_like 'a Hyrax User'

    it "has a display name" do
      expect(user.display_name).not_to be_empty
    end

    it '#to_s yeilds the #display_name' do
      expect(user.to_s).to eq user.display_name
    end

    describe 'roles' do
      it 'is emplty for a new user' do
        new_user = create(:user)
        expect(new_user.roles).to be_empty
      end

      it "#add_role adds a named role" do
        plain_user.add_role('some_role')
        expect(plain_user.roles.inspect).to match(/some_role/)
      end

      it "#add_role creates a new role if needed" do
        expect { plain_user.add_role('named_role') }.to change { Role.count }.by(1)
      end

      it "#add_role uses an existing role if it exists" do
        role = Role.create(name: 'existing_role')
        plain_user.add_role('existing_role')
        expect(plain_user.roles.to_a).to include role
      end

      it "#remove_role removes a named role" do
        new_user = create(:user)
        new_user.add_role('new_role')
        expect { new_user.remove_role('new_role') }.to change { new_user.roles.count }.from(1).to(0)
      end

      it "#remove_role does nothing on non-existant names" do
        existing_roles = plain_user.roles
        plain_user.remove_role('nonexistent_role_name')
        expect(plain_user.roles).to match existing_roles
      end
    end

    describe '#admin?' do
      it 'is false for regular user' do
        expect(user).not_to be_admin
      end

      it 'is true when a user has the "admin" role' do
        admin_user = create(:user)
        admin_user.add_role(:admin)
        expect(admin_user).to be_admin
      end
    end
  end
  ```
### 3. Install devise_ldap_authenticatable
Follow the [usage instructions for devise_ldap_authenticatable](https://github.com/cschiewek/devise_ldap_authenticatable#usage). However, you don't need to run the rails generators for devise again, since you already did that when setting up hydra-role-management. Your tests should still run and pass after this installation, but logging in won't work because it will tell you it can't find an LDAP server.

### 4. Install ladle
1. Add ladle to your gemfile and run bundle install:
  ```
    gem ladle
  ```
1. Make an ldap users file in `config/ldap_data_dev.ldif`:
  ```
  version: 1
  dn: ou=people,dc=example,dc=org
  objectClass: organizationalUnit
  objectClass: top
  ou: people
  dn: ou=groups,dc=example,dc=org
  objectClass: organizationalUnit
  objectClass: top
  ou: groups

  dn: uid=test,ou=people,dc=example,dc=org
  objectClass: top
  objectClass: extensibleObject
  objectClass: uidObject
  objectClass: person
  objectClass: organizationalPerson
  objectClass: inetOrgPerson
  cn: Test User
  givenName: Test
  sn: User
  uid: test
  mail: test@example.org
  ou: people
  # password: 'xoddam'
  userpassword: {SHA}WiCnxkOb4kpy16ON7ZC6mD/iqII=
  dn: uid=dca_admin,ou=people,dc=example,dc=org
  objectClass: top
  objectClass: extensibleObject
  objectClass: uidObject
  objectClass: person
  objectClass: organizationalPerson
  objectClass: inetOrgPerson

  cn: DCA Administrator
  givenName: DCA
  sn: Administrator
  uid: dca_admin
  mail: dca.admin@example.org
  ou: people
  userpassword: password
  dn: uid=tisch_admin,ou=people,dc=example,dc=org
  objectClass: top
  objectClass: extensibleObject
  objectClass: uidObject
  objectClass: person
  objectClass: organizationalPerson
  objectClass: inetOrgPerson

  cn: Tisch Administrator
  givenName: Tisch
  sn: Administrator
  uid: tisch_admin
  mail: tisch.admin@example.org
  ou: people
  userpassword: tisch_admin1
  dn: cn=test,ou=groups,dc=example,dc=org
  objectClass: groupOfUniqueNames
  objectClass: top
  cn: test
  uniquemember: uid=test,ou=people,dc=example,dc=org
  ```
1. Create a ladle rake task in `lib/tasks/ladle.rake`:
  ```ruby
  require 'ladle'

  'Start a ladle server'
  task :ladle do
    conf_path = Rails.root.join('config')

    server = Ladle::Server.new(
      port:  3898,
      quiet: false,
      custom_schemas: conf_path.join('tufts_schema.ldif').to_s,
      ldif:           conf_path.join('ldap_data_dev.ldif').to_s
    )

    begin
      puts 'Starting LDAP server on port 3898'
      server.start
      sleep
    rescue Interrupt
      puts 'Stopping server'
    ensure
      server.stop
    end
  end
  ```
