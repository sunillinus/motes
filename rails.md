#Rails For Zombies

##Scopes
http://api.rubyonrails.org/classes/ActiveRecord/Scoping/Named/ClassMethods.html


##Start up

> rails new TwitterForZombies

> rails g scaffold zombie name:string age:integer

> rake db:migrate

> rails s


##N+1 Queries
```ruby
@zombies = Zombie.all
<% @zombies.each do |zombie| %>
	<%= zombie.brain.flavor %> # <- Bad cause the query is run N times
<% end %>>
```

#####^Bad, cause n+1 queries are run:
```sql
SELECT * FROM “zombies”
N x SELECT * FROM “brains” WHERE “zombie_id” = i
```
#####Use include instead:

```ruby
@zombies = Zombies.include(:brain).all
```
#### Only 2 queries:

```sql
SELECT * FROM “zombies”
SELECT * FROM “brains” WHERE “zombie_id” IN (4, 5, 6, 7)
```

##How Rails do Put (Patch) and Delete

```ruby
<%= link_to 'Delete', zombie, method: :delete %>
```

is rendered as -

```html
<a href '/zombies/4' data-method=delete rel='nofollow'>Delete</a>
```

Which unobtrusive js converts to a form with a hidden input named 'method' and value 'delete' -

```html
<form method='post' action='/zombies/4'>
  <input name="_method" value="delete" type="hidden"/>
</form>
```

###Simple Form helper -

```html
<%= form_for @zombie do |f| %>
  <%= f.label :name %>
  <%= f.text_field :name %>
  <%= f.text_area :bio %>
  <%= f.submit %>
<% end %>
```
Other common input helpers -

```
f.check_box
f.radio_button
f.select
```

###Nested resource path short form: [parent, child] -

```html
<%= link_to 'Show', zombie_tweet_path(@zombie, tweet) %>
<%= link_to 'Show', [@zombie, tweet] %>
<%= link_to 'Delete' [@zombie, tweet], method: :delete %>
<%= form_for([@zombie, @tweet]) do |f| %>
.
.
<% end %>

redirect_to [@zombie, @tweet]
```

##Rails Security

###h and raw

if someone inputs
```
<script>alert('Hello')</script>
```
as the body of a tweet, you want to escape it to
```
&lt;script&gt;
```
to make it safe for others viewing the tweet.

Rails 2 you had to use <%= h to do that, in rails 3, it is <%= does it by default.

Use **<%= raw ** to not escape the html -

```
<%= raw tweet.body %>
```

#### div_for (calls dom_id on Activerecord)

Instead of

```html
<%= @tweets.each do |tweet| %>
  <div id=<%= tweet.id %> class='tweet'>
    <%= tweet.body %>
  </div>
<% end %>
```
Do

```
<%= @tweets.each do |tweet| %>
  <%= div_for tweet do %>
    <%= tweet.body %>
  <% end %>
<% end %>
```

Other useful helpers -

- truncate('Long sentence here', :length => 10, :seperator => ' ')
- pluralize(count, "zombie")
- "string".titleize
- array.to_sentence
- time_ago_in_words @zombie.created_at
- number_to_currency
- number_to_human 1234567890123456

##Mailer

> rails g mailer UserMailer welcome_mail bye_mail

```ruby
class UserMailer < ActionMailer::Base
  default from: 'admin@example.com'

  def welcome_mail(user)
    @user = user # so user is available in mail view
    attachments['z.pdf'] = File.read("#{Rails.root}/public/zombie.pdf")
    mail to: user.email, subject: 'Welcome'
  end
end
```

##Asset Tags

```ruby
<img src="/assets/weapon.png" /> => <%= image_tag 'weapon.png' %>
<script src="/assets/weapon.js" /> => <%= javascript_include_tag 'weapon.js' %>
<link href="/assets/weapon.css" media="screen" rel="stylesheet" type="text/css" />
=> <%= stylesheet_link_tag 'weapon.css' %>
>
```

##Response Formats

```ruby
def show
  @zombie = Zombie.find(params[:id])
  respond_to do |format|
    format.html # render show.html.erb
    format.json {json: @zombie}
  end
end
```
format can also take a block -
```ruby
format.html do
  render :dead if @zombie.dead
end
```
To render just json-
```ruby
render json: @zombie
```
For JSON create, best practice to set HTTP header location (url) of the new resource -
```
render json: @zombie, status: :created, location: @zombie
```
Errors:
```
redner json: @zombie.errors, status: :unprocessable_entity
```

##Custom Routes
```ruby
resources :zombies do
  get :decomp, on: :member
  post :search, on: :collection
end

<%= form_tag search_zombie path do |f| %>
>
```
##Customizing JSON
```ruby
@zombie.to_json(:only => [:name, :age])
@zombie.to_json(:except => [:bio])
@zombie.to_json(:include => [:brain])
```

To set default JSON for a model

```ruby
class Zombie < ActiveRecord::Base
  def as_json(options={})
    super(options || {include: => :brain, except: [:bio, :created_at]})
  end
end

# multi-level as_json
def as_json(options=nil)
  super(options || {:only => :name, :include => {:weapons => {:only => [:name, :ammo]}}})
end

```


##Ajaxifying

###2 Ways - JS and JSON

####JS way:

3 Steps

1. Set link/form **remote** attribute to true:
```ruby
<%= link_to 'Delete', @zombie, method: :delete, remote: true %>>

<%= form_for(Zombie.new, remote: true) do |f| %>>
```

2. Make the action accept Ajax request (format.js)
```ruby
ZombiesController#destroy:
respond_to |format| do
    format.js
end
```
3. Add the js template:
```js
destroy.js:
$('#<%= dom_id(@zombie) %>').fadeOut(); # assuming div_for(@zombie)
create.js:
<% if @zombie.new_record? %>
  $('input#zombie_name').effect('highlight', {color: 'red'});
<% else %>
  $('div#zombies').append("<%= escape_javascript(render @zombie) %>");
  $('div#<%= dom_id(@zombie) %>').effect('highlight'); // must require jquery_ui
<% end %>
```

###Refactoring rails view loops
```html
<div id='zombies'>
  <%= @zombies.each do |z|
    <%= render z %>  <!-- uses classname of the object to render '_zombie.html.erb' -->
  <% end %>
</div>
```
^Can further refactor the loop as:
```html
<div id='zombies'>
  <%= render @zombies %>
</div>
```

##JSON API Way

1. Normal form/link.
2. JS that intercepts the submit/click and makes AJAX request.
3. Action returns JSON.
4. AJAX request callback that accepts the JSON response and modifies the view.

```js
$(document).ready ->
  $('div#custom_phase2 form').submit (event) ->
    event.preventDefault() // prevent default
    url = $(this).attr('action') // grab the form url
    custom_decomp = $('div#custom_phase2 #zombie_decomp').val() // grab the value to submit
    $.ajax // make ajax req
      type: 'put'
      url: url
      data: { zombie: { decomp: custom_decomp } }
      dataType: 'json'
      success: (json) ->
        $('#decomp').text(json.decomp).effect('highlight')
        $('div#custom_phase2').fadeOut() if json.decomp == "Dead (again)"
```

One more -

coffee:
```js
$(document).ready ->
  $('div#reload_form form').submit (event) ->
    event.preventDefault()
    url = $(this).attr('action')
    ammo = $('#ammo_to_reload').val()
    $.ajax
      type: 'put'
      url: url
      data: { ammo_to_reload: ammo }
      dataType: 'json'
      success: (json) ->
        $('#ammo').text(json.ammo).effect('highlight')
        $('#reload_form').fadeOut() if json.ammo >= 30
```

controller:

```ruby
class WeaponsController < ApplicationController
  def reload
    @weapon = Weapon.find(params[:id])

    respond_to do |format|
      if @weapon.ammo < 30
        @weapon.reload(params[:ammo_to_reload])

        format.json { render :json => @weapon.to_json(:only => :ammo), status: :ok }
        format.html { redirect_to @weapon, notice: 'Weapon ammo reloaded' }
      else
        format.json { render :json => @weapon.to_json(:only => :ammo), status: :unprocessable_entity }
        format.html { redirect_to @weapon, notice: 'Weapon not reloaded' }
      end

      format.js
    end
  end
end
```
html:
```html
<p id="notice"><%= notice %></p>
<ul>
  <li>
    <em>Name:</em>
    <%= @weapon.name %>
  </li>
  <li>
    <em>Condition:</em>
    <span id="condition"><%= @weapon.condition %></span>
    <%= link_to "Toggle", toggle_condition_user_weapon_path(@user, @weapon), remote: true %>
  </li>
  <li>
    <em>Ammo:</em>
    <span id="ammo"><%= @weapon.ammo %></span>
  </li>
</ul>

<div id="reload_form">
<%= form_for [@user, @weapon], url: reload_user_weapon_path(@user, @weapon), remote:true do |f| %>
  <div class="field">
    Number of bullets to reload:
    <%= number_field_tag :ammo_to_reload, 30 %> <br />
    <%= f.submit "Reload" %>
  </div>
<% end %>
</div>

<%= link_to 'Edit', edit_weapon_path(@weapon) %> |
<%= link_to 'Back', weapons_path %>
```
