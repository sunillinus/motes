#Rails (3.2) App From Scratch

1. rails new <app> -d postgresql
2. git init
3. git add .
4. git commit -m "new app"
5. gem 'twitter-bootstrap-rails' # in Gemfile
6. bundle install
7. rails g bootstrap:install
8. rails g bootstrap:layout application (will overwrite application.html.erb)
9. rails g scaffold <model> <field_1>:<type> <field_2>:<type>
10. rake db:create
11. rake db:migrate
12. rails s

#Devise

1. gem 'devise'
2. bundle install
3. rails g devise:install
4. rails g devise User
5. rake db:migrate


#### Devise Helper methods

- user_signed_in?
- current_user

#### Routes
- new_user_session (log in)
- destroy_user_session (log out)
- new_user_registration (sign up)
- edit_user_registration

####Access Control
before_filter :authenticate_user!

#Rails App Templates

1. Write a ruby 'script' file with instruction to setup the app
2. rails new my_app -m <script file>
