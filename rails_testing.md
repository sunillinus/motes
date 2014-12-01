#Rails Testing

###Testing Philosophies
- No Testing
- Verification Testing (test to verify the functionalty after you implemented it)
- Test First (Write test that fails, write code to make test pass)
- Test Driven Design (Describe behavior with design)

###TDD Paradigm - Red -> Green -> Refactor

##Basic Test

><subject>_test.rb
```ruby
require 'test/unit'
class <Subject>Test < Test::Unit::TestCase
  def test_<what_we_are_testing>
    <Assertion> # Good practice to limit assertions to 1/test
  end
end
```

###Assert methods

> assert <boolean conditional>

> assert_equal <expected> <method/value being tested>

> assert_nil <object>, assert_not_nil

> assert_match(<regex>, <string>), assert_no_match

> assert_raise(error) {block}

>assert_kind_of <Class>, <object>


### Rake tasks

- rake db:test:prepare => Check for migrations + load schema
- rake test or just rake => run db:test:prepare + all tests
- rake -Itest test/unit/zombie_test.rb => run tests in one file
- rake -Itest test/unit/zombie_test -n test_the_truth => run single test

*Need -Itest to fine test_helper.rb

### Model Test

>test/unit/zombie_test.rb
```ruby
require 'test_helper'
class ZombieTest < ActiveSupport::TestCase
  def test_name_validation
    z = Zombie.new
    assert !z.valid?, 'Name is not being validated'
  end
end
```

>Alternative form, thanks to ActiveSupport::TestCase
```ruby
require 'test_helper'
class ZombieTest < ActiveSupport::TestCase
  test "name validation" do
    z = Zombie.new
    assert !z.valid?, 'Name is not being validated'
  end
end
```

---

## Fixtures

> test/fixtures/zombies.yml

```
ash:
  id: 1
  name: ash
  status: dead

tim:
  id: 2
  name: tim
  status: rotting
```
To load:

zombies(:ash)


>Testing ownership
```ruby
test "should contain only tweets that belong to zombie" do
  z = zombies(:ash) # load from fixture
  assert z.tweets.all? {|t| t.zombie == z}
end
```

###setup method

setup method is executed before every test.

```ruby
  def setup
    # use instance var to make it available in the test methods
    # in the class
    @tweet = tweets(:hello_world)
  end
```

>Custom assert for validations
```ruby
  def assert_attribute_is_validated(model, attribute)
    # This line sets the specified attribute to nil
    model.assign_attributes(attribute => nil)
    assert !model.valid?, "#{attribute.to_s} is not being validated"
  end
```

---

##Mocks and Stubs

```ruby
group :test do
  gem 'mocha'
end
```

#####Stubs replace a method with code that returns the specified value.

#####Mocks (expects) are stubs that check if the method is being called (with a hidden assert).

```ruby
@zombie.weapon.stubs(:slice) # Stub
@zombie.weapon.expects(:slice) # Mock

@zombie.weapon.expects(:slice).with(@zombie.strength)
  .returns true
```

To stub object

```ruby
loc = stub(latitutde: 2, longitude: 3)
```
---

##Integration Testing

Tests the full application stack (but not web server). From rack to rails and back.

Why no controller/view tests - cause there shouldn't really be any logic in those
and integration tests should cover whatever it needs.

Low level helpers
- get zombies_path
- post zombies_path zombie: {name: 'ash'}
- patch zombies_path(zombie) zombie: {name: 'ash'}
- delete zombies_path(zombie)
- follow_redirect!

- assert_response
- assert_redirected_to <url>
- assert_tag 'a', attributes: {href: <url>}
- assert_no_tag 'div', attributes {class: 'content'}
- assert_select 'h1', <content>

### Gen Integration test
>rails g integration_test zombies =>
>test/integration/zombies_test.rb

```ruby
require 'test_helper'
class ZombiesTest < ActionDispacth::IntegrationTest
  fixtures :all # load all fixtures

  test 'show zombie page' do
    z = zombies(:ash)
    get zombie_path(z)

    # multiple asserts okay in integration test since
    # setup cost is higher
    assert_response :success
    assert_select 'h1', zombie.name
  end
end
```

## Capybara

-Behaves more like browser.

To use capybara:

```ruby
group :test do
  gem 'capybara'
end
```

>add to test/test_helper.rb

```ruby
require 'capybara/rails'

class ActionDispatch::IntegrationTest
  include Cpaybara::DSL

  def teardown
    Capybara.reset_sessions!
    Capybara.use_default_driver
  end
end
```
### Capybara Helper Methods
- visit

Arg can be text or element id:
- click_link 'Link'
- click_button 'Button'
- click_on 'Link or Button'

Arg can be form label or element name or element id
- fill_in 'Text Box', with: 'Name'
- check 'Check Box'
- uncheck 'Check Box'
- select 'Option', from: 'Select Box'
- choose 'A Radio Button'
- attach 'Image', 'path/to/image.jpg'

- current_path
- current_url

- has_content?
- has_no_content?
- has_selector?
- has_no_selector?
- has_link?
- has_field?
- has_css?
- has_xpath?
- within

>Examples:
```ruby
within '#zombie_1' do
  has_content? 'Ash'
end
has_selector? '#zombie_1 h1', text: 'Ash'
has_selector? '.zombies', count: 5, visible: true
```

```ruby
test 'sign up page' do
  visit root_url
  click_link 'Sign Up'
  fill_in 'Name', with: 'Rhino'
  click_button 'Sign Up'
  assert_equal user_path(User.last), current_path, 'Sign up failed!!'
end
```
---

## Factories

Fixtures has some drawbacks:

- Associations are hard
- Fixtures files grow exponentially with edge cases
- Lot of repetitions

```ruby
group :development, :test do
  gem 'factory_girl_rails'
end
```

>test/factories/zombies.rb
```ruby
FactoryGirl.define do
  factory :zombie do
    name 'MyString'
    graveyard 'MyString'
  end
end
```

-
>To use:
```ruby
- Factory(:zombie) # same as FactoryGirl.create(:zombie)
- FactoryGirl.build(:zombie)
- FactoryGirl.attributes_for(:zombie)
- Factory(:zombie, name: 'Ash1') # to overwrite default name
```

-
>Sequence:
```ruby
FactoryGirl.define do
  factory :zombie do
    sequence(:name) {|i| "Ash_#{i}"}
    graveyard: 'Sunnyvale'
  end
end
```
100.times { factory(:zombie) }

>Associations
```ruby
FactoryGirl.define do
  factory :weapon do
    name 'Chainsaw'
    association :zombie
  end
end
```

To custom name a factory, pass the class or nest it within the parent factory:

```ruby
FactoryGirl.define do
  factory :projectile_weapon, class: Weapon do
    name 'Uzi'
    association :zombie
  end
end

# Or

FactoryGirl.define do
  factory :weapon do
    name 'Chainsaw'

    factory :projectile_weapon do
      name 'Uzi'
    end
  end
end
```

Testing with factory

```ruby
test 'decapitate should set status to dead again' do
  zombie = FactoryGirl.build(:zombie, status: 'dead')
  zombie.decapitate
  assert_equal 'dead again', zombie.status
end

test 'show zombie tweet page' do
  tweet = Factory(:tweet)
  zombie = tweet.zombie
  visit tweet_url(tweet)
  assert_equal tweet_path(tweet), current_path
end
```
