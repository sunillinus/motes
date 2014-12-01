#RSpec

Uses BDD terminology

1. gem install rspec
2. rspec --init (in proj dir)

Within Rails:

```ruby
group :development, :test do
  gem 'rspec-rails'
end
```
2. bundle install
3. rails g rspec:install => spec/spec_helper.rb

To run:

```
rspec
rspec spec/models/
rspec spec/lib/zombie_spec.rb
rspec spec/lib/zombie_spec.rb:4
```

add to .rspec
```
--color
--format documentation
```

>spec/lib/zombie_spec.rb
```ruby
require 'spec_helper'
require 'zombie'
describe Zombie do
  it 'is named Ash' do
    zombie = Zombie.new
    zombie.name.should == 'Ash' # expectation
  end
  it 'has no brains' do
    zombie = Zombie.new
    zombie.brains.should < 1
  end
end
```


>Modifiers and Matchers
```ruby
should be_true #should == true
should be_false
should be < 1 # should < 1
should_not == 1
zombie.should be_hungry # same as zombie.hungry?.should == true
```

<predicate_method>?.should == true => should be_<predicate_method>
<predicate_method>?.should == false => should_not be_<predicate_method>

To make a test pending:
1. leave the body out
2. xit
3. put 'pending' in the body

## RSpec Matchers

```ruby
zombie.should_not be_valid
zombie.name.should match(/Ash \d/) # match
zombie.tweets.shound include(tweet1) # include
zombie.should have(2).weapons # have, same zombie.weapons.count.should == 1
zombie.should have_at_least(2).weapons
zombie.should have_at_most(2).weapons
zombie.should respond_to(:hungry?)
width.should be_within(0.1).of(33.3)
zombie.should exist
hungry_zombie.should be_kind_of(Zombie)
status.should be_an_instance_of(String)
```

###expect {}.to Block

#### change
```ruby
zombie = Zombie.new(name: 'Ash')
expect { zombie.save }.to change { Zombie.count }.by(1) # change block is run before and after expect block
```
Can also do from() and to() also chain them from(1).to(5)

#### raise_error
```ruby
zombie = Zombie.new
expect { zombie.save! }.to raise_error(ActiveRecord::RecordInvalid)
```
to, not_to and to_not

## DRYing Specs

```ruby
describe Zombie do
  it 'responds to name' do
    zombie = Zombie.new
    zombie.should respond_to(:name)
  end
end

describe Zombie do
  it 'responds to name' do
    subject.should respond_to(:name) #subject = Zombie.new
  end
end

describe Zombie do
  it 'responds to name' do
    should respond_to(:name) # subject is implicit
  end
end

describe Zombie do
  it 'responds to name' { should respond_to(:name) } # using {}
end

describe Zombie do
  it { should respond_to(:name) } # remove descriptive string
end

describe Zombie do
  it { subject.name.should == 'Ash' }
end

describe Zombie do
  its(:name) { should == 'Ash' } # its(:name) => subject.name
end
```

it 'blah' do
  subject.<method>.should == <value>
end

it { subject.<method>.should == <value> }

its(:<method>) { should == <value> }

#### context
```ruby
describe Zombie do
  context 'when hungry' do
    it 'blah'
    context 'with a veggie pref' do
      subject { Zombie.new(veggie: true) }
      it 'blah'
      it 'blah'
    end
  end
end

```

#### let
```ruby
let(:axe) { Weapon.new(name: 'axe') }
```
Also do

```ruby
subject(:axe) { Weapon.new(name: 'axe') }
```

let!(:zombie) { Zombie.create }

will create zombie before every example, otherwise it is lazy evaluated


###Filters

before {} (same as before(:each) {}),
before(:all)
after(:each),
after(:all)

###Shared examples

>spec/models/zombie_spec.rb
```ruby
describe Zombie do
  it_behaves_like 'the undead'
end
```

>spec/support/shared_examples_for_undead.rb
```ruby
shared_examples_for 'the undead' do
  its(:pulse) { should be_false }
end
```


To not use implicit subject

>spec/models/zombie_spec.rb
```ruby
describe Zombie do
  it_behaves_like 'the undead', Zombie.new
end
```

>spec/support/shared_examples_for_undead.rb
```ruby
shared_examples_for  'the undead' do |undead|
  undead.pulse.should be_false
end
```

###Tags

Can use tags to run examples (tests) selectively

>rspec --tag vegan spec/models/zombie_spec.rb
```ruby
it 'prefers vegan brains', vegan: true
```

Use ~ to not run examples with the tag

>rspec --tag ~vegan spec/models/zombie_spec.rb


##Mocks and Stubs

Example

```ruby
describe Tweet do
  context 'after create' do
    let(:zombie) { Zombie.create(email: 'anything@example.org') }
    let(:tweet) { zombie.tweets.new(message: 'Arrrrgggghhhh') }
    let(:mail) { stub(:mail, deliver: true) }

    it 'calls "tweet" on the ZombieMailer' do
      ZombieMailer.should_receive(:tweet).with(zombie, tweet).and_return(mail)
      tweet.save
    end

    it 'calls "deliver" on the mail object' do
      # can also do ZombieMailer.stub(tweet: mail)
      ZombieMailer.stub(:tweet).and_return(mail)
      mail.should_receive(:deliver)
      tweet.save
    end
  end
end
```

Stub options

target.should_receive(:function).once
.twice
.exactly(3).times
.at_least(2).times
.at_most(3).times
.any_number_of_times
target.should_receive(:function).with(no_args())
.with(any_args())
.with("B", anything())
.with(3, kind_of(Numeric))
.with(/zombie ash/)

###Custom Matcher

>spec/support/validate_numericality_of.rb

```ruby
module ValidateNumericalityOf
  class Matcher
    def initialize(attribute)
      @attribute = attribute
      @options = {}
      @errors = []
    end

    def matches?(model)
      @model = model
      @model[@attribute] = "not a number"
      @model.valid?

      if !@model.errors[@attribute].include?("is not a number")
        @errors << "numericality"
      end

      if @options[:only_integers]
        @model[@attribute] = 1.5
        @model.valid?
        if !@model.errors[@attribute].include?("must be an integer")
          @errors << "as an integer"
        end
      end

      @errors.empty?
    end

    def only_integers
      @options[:only_integers] = true
      self
    end

    def failure_message
      "#{@model.class} failed to validate: #{@attribute} #{@errors.join(', ')}"
    end

    def negative_failure_message
      "#{@model.class} unexpected validation: #{@attribute} #{@errors.join(', ')}"
    end

    def description
      "validate numericality of #{@attribute}"
    end
  end

  def validate_numericality_of(attribute)
    Matcher.new(attribute)
  end
end

RSpec.configure do |config|
  config.include ValidateNumericalityOf, type: :model
end
```

>spec/models/zombie_spec.rb
```ruby
describe Zombie do
  subject { Zombie.new }
  it { should validate_numericality_of(:iq) }
end
```


- it { should validate_presence_of(:name).with_message('oh noes') }
- it { should ensure_inclusion_of(:age).in_range(18..25) }
- it { should have_many(:weapons).dependent(:destroy) }
- it { should have_many(:weapons).class_name(OneHandedWeapon) }
