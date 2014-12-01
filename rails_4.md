<!-- TOC depth:6 withLinks:1 updateOnSave:1 -->
- [Rails 4: Zombie Outlaws](#rails-4-zombie-outlaws)
	- [Concerns](#concerns)
	- [Thread Safety](#thread-safety)
	- [Rails 4 ActiveRecord Finders](#rails-4-activerecord-finders)
		- [ActiveModel::Model](#activemodelmodel)
	- [Rails 4 Whitelisting Parameters](#rails-4-whitelisting-parameters)
		- [Filters](#filters)
	- [Session Cookies](#session-cookies)
	- [Custom Flash Types](#custom-flash-types)
	- [Rails 4 View Helpers](#rails-4-view-helpers)
	- [Rails 4 Tests](#rails-4-tests)
	- [ETags](#etags)
		- [Memcache](#memcache)
	- [Fragment Caching and Russian Doll Caching](#fragment-caching-and-russian-doll-caching)
		- [Nested fragments](#nested-fragments)
	- [Rails Live Streaming/Updating](#rails-live-streamingupdating)
	- [TurboLinks](#turbolinks)
<!-- /TOC -->#Rails 4: Zombie Outlaws
---

In rails 3 routes 'match' matches any HTTP method, making XSS attacks possible.

```ruby
match '/items/:id/purchase', to: 'items#purchase' # accepts any http method including GET

<a href="http:yourapp.com/items/2/purchase">Click</a>
```

In rails4, use 'post' instead of 'match' or 'via: post' (via: all for all)

##Concerns

Use concerns to group resources that are usually nested together (so that DRY nesting).

```ruby
concern :sociable do
  resources :comments
  resources :tags
end

resource(:messages, :concerns => :sociable)
resource :posts, concerns: :sociable
```
Passing Options

```ruby
concern :sociable do |options|
  resources :comments, options
  resources :tags, options
end

resources :messages, concerns: :sociable
resources :posts do
  concerns :sociable, only: :create
end
```

>Can move concerns to a diff class (that responds to self.call)
```ruby
concern :sociable Sociable
resources :messages, concerns: :sociable
resources :posts, concerns: :sociable
```

>apps/concerns/sociable.rb
```
class Sociable
  def self.call(mapper, options)
    mapper.resources :comments, options
    mapper.resources :tags, options
  end
end
```

##Thread Safety
Rails 3 thread safety is **disabled** by default in **production**

>config/environments/production.rb
```ruby
MyApp::Application.configure do
  #config.threadsafe! <- commented out for production. Deprecated in rails 4
end
```

In Rails 4 thread safety is **enabled** by default on -

>config/environments/production.rb
```ruby
MyApp::Application.configure do
  config.cache_classes = true # prevents class reloading between requests and
                              # makes sure Rack:Lock is not included in middleware stack
  config.eager_load = true # loads all code before new threads are created
end
```

##Rails 4 ActiveRecord Finders

| Old | Rails 4 |
|-----|-----|
| Post.find(:all, :conditions {author: 'admin'}) |  Post.where(author: 'admin')
| Post.find_all_by_title('Rails 4') | Post.where(title: 'Rails 4') |
| Post.find_last_by_title('Rails 4') | Post.where(title: 'Rails 4').last |
| Post.find_by_title('Rails 4') <- still works | Post.find_by(title: 'Rails 4') |
| Post.find_by_title('Rails 4', conditions: {author: 'admin'}) | Post.find_by(title: 'Rails 4', author: 'admin') |
| Post.find_or_create_by_title('Rails 4') | Post.find_or_create_by(title: 'Rails 4') |
| Post.find_or_initialize_by_title('Rails 4') | Post.find_or_initialize_by(title: 'Rails 4') |
| @post.update_attribute(:title, 'Rails 4') | @post.update_columns(post_params) |
| @post.update_column(:title, 'Rails 4') | -- same -- |
| @post.update_attributes(post_params) | @post.update(post_params) |
| Tweet.scoped | Tweet.all # returns ActiveRecord::Relation |
| scope :sold, wher(state: 'sold') # eager evaluated scopes are deprecated | scope :sold, -> { where(state: 'sold') } |
| default_scope where(state: 'available') | default_scope { where(state: 'available') } *or* default_scope ->{ where(state: 'available') } |
| Post.where('author != ?' author ) # won't work if author is nil | Post.where.not(author: author) |
| Post.includes(:comments).where("comment.name = 'foo'") | Post.includes(:comments).where("comments.name = 'foo'").refrences(:comments) # msut explicitly tell when refrenceing another table from a string |
| Post.order('created_at DESC') | Post.order(created_at: :desc) |

>Problem with Rails 3 style eagerly evaluated scopes is the scope is resovled once on class load, so date won't change
```ruby
scope :recent, where(published_at: 2.weeks.ago)
```

find_by just wraps where, so accepts same args as where -

>activerecord/lib/active_record/relation/finder_methods.rb
```ruby
def find_by(*args)
  where(*args).take
end
```
Post.find_by("published_on < ?", 2.weeks.ago)


Can also use Post.where(:title => 'Rails 4').find_or_create instead of find_or_create_by(:title => 'Rails 4') but any callbacks exeute within the scope of the where.

Post.none # returns ActiveRecord::Relation but never hits the DB

Post.where('author != ?' nil ) will gen wrong SQL -> SELECT "post".* FROM "posts" WHERE(author != NULL)

Post.where.not(author: nil) will gen right SQL -> SELECT "posts".* FROM "posts" WHERE(author is NOT NULL)

###ActiveModel::Model
can mixin with any Ruby class to make it behave like ActiveRecord even without DB table

##Rails 4 Whitelisting Parameters

In Rails 3 models whitelisted paramerts using **attr_accessible**.
This controlled what params could mass assigned to a model using update_attributes

In Rails 4 controllers do it using **params.require().permit()**.

```ruby
params.require(:user).permit(:name)
```
It also checks for param types.

Can still use attr_asseccible in Rails 4, but have toinstall **gem protected_attributes**

###Filters
Filters are now called actions (but filters still work so whatever)

##Session Cookies

Rails use session cookie store by default. So session is stored in the client browser
as a cookie which the browser send back.

Initial login

>controllers/sessions_controller.rb
```ruby
def create
  session[:user_id] = user.id
end
```

Subsequnt requests
>controllers/application_controller.rb
```ruby
def current_user
  current_user ||= User.find(session[:user_id])
end
helper_method :current_user
```
In rails 3, the cookie is digitally signed, so can't be altered, but can be read.

To read -
```ruby
require 'rack'
cookie = "blah"
Rack::Session::Cookie::Base64::Marshal.new.decode(cookie)
```

In Rails 4, the cookie is also encrypted using the secret_key_base (so make sure secret keybase is not checked into the source code)

>config/initializers/secret_token.rb
```ruby
MyApp.Application.config.secret_key_base = ENV['SECRET_KEY_BASE']
```

##Custom Flash Types

You can register cutom flash types by

>controllers/application_controller
```ruby
class ApplicationCOntroller < ActionController::Base
  add_flash_types :grunt, :snarl, :woof
end
.
.
.
redirect_to @user, grunt: 'Grrrr...'
.
.
.
<div id='grunt'><%= grunt %>>
```

##Rails 4 View Helpers

In addtion to collection_select, there is collection_radio_buttons and collection_check_boxes

```ruby
collection_radio_buttons(:object, :method, :collection, :value_method, :text_method)
collection_radio_buttons(:item, :owner_id, Owner.all, :id, :name)
```

>Date filed helper
```ruby
<%= f.date_select :return_date %>> # old
<%= f.date_field :return_date %>> # generates HTML5 date type
```

Rails 4 has ruby template to gen JSON

>index.json.ruby
```ruby
owner.hashes = owner.map do |owner|
  { name: owner.name, url: owner_url(owner) }
end
owner_hashes.to_json
```

##Rails 4 Tests

rake test:models
rake test:controllers
.
.


##ETags

ETags (Entity Tags) is used to check if a page has changed since last time.
Rails 3 and 4 by default use ETags.
>Here is how ETags work -
1. Client requests a page for first time.
2. The Rails app renders the body
3. The Rails app (controller?) sets a ETag header field to the MD5 checksum of the body: headers['ETag'] = Digest::MD5.hexdigest(body)
4. Rails sends the body and the ETag header in the response back to the client
5. The client caches the response and the ETag
6. When the client requests the same page again, it send the ETag associated with the page in the headers['If-None-Match'] field.
7. The Rails app renders the body again and calculates the ETag (MD5 checksum of the body) again the same way.
8. If the two ETags match, the body is not send bacl to the client, instead client gets HTTP status code 304 (Not Modified).
9. The client on getting 304, will load the page from the cache.

^The problem is the server still has to render the whole page

In Rails 3&4 you can set custom ETags (instead of just the body) using fresh_when
```ruby
fresh_when(@item)
```

>fresh_when does two things:
1. Sets the ETag to Digest::MD5.hexdigest(@item.cache_key). cache_key is `<model_name>/<id>-<updated_at>`
2. If the ETag is same as the one 'If-None-Match' header in the request, it will send 304 back.

This way the server will not have to render the whole body unless something changed.

To add multiple dependecies - fresh_when([@item, current_user.id])


In Rails 4, you can declare etag at controller level -

```
etag { current_user.id}
etag { current_user.age }
```

### Memcache

Rails 4 uses Dalli for accessing mamcached servers

gem 'dalli'

>config/environments/production.rb
```ruby
config.cache_store = :mem_cache_store
config.action_controller.perform_caching = true
```

Cache Store API
```ruby
Rails.read('key')
Rails.write('key', value)
Rails.fetch('key') { value }
```

##Fragment Caching and Russian Doll Caching

>app/vies/comments/_comment.html.erb
```html
<% cache comment do %>
  <li><%= comment %></li>
<% end %>
```

This will check the server side cache for comment.cache_key (`<model name>/<id>-<updated_at>`).
If found it will return the html fragment with that cache-key, else will gen the
html fragment and return it (and store it with that cache-key).

If the cache-key is <model name>/<id>-<updated_at>, it won't work if the html
partial itself changes. So you have to add a version string like this -

```html
<% cache ['v1', comment] do %>
  <li><%= comment %><li>
<% end %>

<% cache ['v2', comment] do %>
  <li><%= comment %> - <%= comment.author %><li>
<% end %>
```

**Rails 4** takes care of this automatically by adding MD5 hash of the html fragment to the
cache-key (<model name>/<id>-<updated_at>/<md5 of template>)

###Nested fragments


```html
>app/views/projects/show.html.erb
<%= render @project.documents %>

>app/views/documents/_document.html.erb
<% cache document do %>
  <h3><%= document.title %></h3>
  <ul>
    <%= render document.comments %>
  </ul>
<% end %>

>app/views/comments/_comment.html.erb
<% cache comment do %>
  <li><%= comment %></li>
<% end %>
```

To invalidate the document cache if a comment is updated,
use **touch: true** in the relationship

```ruby
class Comment < ActiveMode::Base
  belongs_to :document, :touch => true # will also 'tocuh' parent document when a comment is modified
end
```

For comment template changes, in Rails 4 the cache-key includes the MD5 of child
templates as long as the partials are explicitly rendered.

```
<% = render partial: "comments/comment", collection: document.comments %>
```

Won't work with utility methods or scopes. For those add this comment syntax

```
<%# Template Dependency: comments/comment %>
<%= render document.recent_comments %>
```

##Rails Live Streaming/Updating

To Live stream from Rails:

1. ** include ActionController::Live** in your controller
```ruby
class ItemsController
  include ActionController::Live
end
```

2. Use a compatible server like Puma (WEBrick buffers output and that breaks streaming)
```
gem 'puma'
```

3. In your action, set content-type header to 'text/event-stream'
  `response.headers['Content-Type'] = 'text/event-stream' # Must happen before you start streaming`

4. Write to the response stream object.
  `response.stream.write "Hello..."`

5. Close the stream!
  `response.stream.close`

6. Write JS using the Event Source lib to read from the server and modify the view

```ruby
def show
  response.headers['Content-Type'] = 'text/event-stream' # Must happen before you start streaming
  3.times do
    response.stream.write "Hello..."
    sleep 1
  end
  response.stream.close
end
```

```js
$(document).ready(initialize);
function initialize() {
  var event_source = new EventSource('/items/events');
  event_source.addEventListener('message', update);
}

function update(event) {
  var item = $('<li>').text(event.data);
  $('ul#items').append(item);
}
```

##TurboLinks
Rails 4 uses turbolinks by default.

With out turbolinks, when you click on a non-ajax link, whole page refreshes.
Even with ETags, the new page has to be completely redrawn by the browser.

With Turbolinks, when a link is clicked, it uses AJAX to get the new page and
updates just the title and the body. None of the assets - CSS/JS is reloaded.

If data-turbolinks-track option is set to true for an asset, TurboLinks will
track that asst and ask the browser to do a full reload if that asset has changed

```html
<%= javascript_include_tag 'application', "data-turbolinks-track"=>true %>
```

Problem with Turbolink - DOM ready event *$(document).ready()) won't fire after
the first page load.
To fix this also have to do $(document).on('page:load', foo).

Also can include gem 'jquery-turbolinks' (right it after jquery in application.js).
This will fire DOM ready event after a 'page:load'

Or

Bind events straight to the document -
```js
$(document).on('click', '#link', foo);
```

Use page:fecth and page:change evnets to show/hide a loader:
```js
$(document).on('page:fetch', function() {
  $('#loader').show();
});

$(document).on('page:hide', function() {
  $('#loader').hide();
});
```

Use **data-no-turbolink="true"** attribute to turn turbolinks selectively for individual links
or a whole container.
