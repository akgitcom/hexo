title: "Tracking Devise User Logins & Sign-ups"
date: 2015-01-31 11:48:58
tags: [devise,logs,tracking,用户日志]
---

link:[http://joanswork.com/devise-usage-tracking/](http://joanswork.com/devise-usage-tracking/)
##Tracking Devise User Logins & Sign-ups

I need to track the user activity being handled by the [Devise](https://github.com/plataformatec/devise) gem. A set-up that’s lean, specific to Devise and easily limited to only logins for production. The following is what I came up with. The [impressionist](https://github.com/charlotte-ruby/impressionist) gem would have been an option but it’s a little much for my needs.

I’m working with Rails 4.0.2 / Ruby 2.0.0 / Devise 3.2.2. The full source code can be found in [jehughes/rails4-example](https://github.com/jehughes/rails4-example).

##The first question is where to put the hooks into Devise?

*(In other words, is this possible without creating a mess?)*

I decided to use the ‘after’ action methods that are provided by Devise. The methods you override to redirect the user after they login, logout, etc.

For example, in **app/controllers/local_devise/registrations_controller.rb** :

```ruby
def after_update_path_for(resource)
  DeviseUsageLog.log(resource, DeviseAction::Edit)
  root_url
end
```

We’ll define DeviseUsageLog and DeviseAction later.

Here’s a list where the logging was added in the same way as the method above:

file | method
----|------
<font color="green">applications_controller.rb</font> | after_sign_in_path_for  
<font color="green">local_devise/unlocks_controller.rb</font> | after_unlock_path_for  
<font color="green">local_devise/passwords_controller.rb</font> | after_resetting_password_path_for 
<font color="green">local_devise/confirmations_controller.rb</font> | after_confirmation_path_for, after_confirmation_set_password_path_ for
<font color="green">local_devise/registrations_controller.rb</font> | after_inactive_sign_up_path_for, after_sign_up_path_for, after_- update_path_for

In addition, to log the deletion of a user account you’ll need to override the *destroy* method in **local_devise/registrations_controller.rb** :
```ruby
def destroy
  DeviseUsageLog.log(resource, DeviseAction::Delete)
  super
end
```

I’m not tracking user logouts but that would be **after_sign_out_path_for** in **applications_controller.rb**. (Probably. Haven’t tried it out.)

##Use a DeviseActions enum to keep things tidy

On my wish list was avoiding vague hardcoded strings like ‘edit’ or ‘new’ scattered throughout the source.

I used the [classy_enums gem](https://github.com/beerlington/classy_enum) to enforce a list of valid action names.

After installing the gem, use the generator to create the enum:

```ruby
rails g classy_enum DeviseAction new confirmed login password unlocked edit delete
```

*(Yeah, probably overkill but it keeps things in the source and database tidy.)*

##Next create somewhere to log the data

**Create the model** DeviseUsageLog.

```ruby
  create_table "devise_usage_logs", force: true do |t|
    t.integer  "user_id",    null: false
    t.string   "user_ip"
    t.string   "role"
    t.datetime "created_at"
    t.datetime "updated_at"
    t.string   "username"
    t.string   "action"
  end
```

**action** is the DeviseAction enum. Make the connection between the model and enum by adding the following to **model/devise_usage_log.rb**:

```ruby
classy_enum_attr :action, enum: 'DeviseAction', allow_nil: true
```

Allow **action** to be null since we don’t want potential problems to impact the user. This is the same reason the User and DeviseUsageLog models have not been connected through a ‘belongs_to’ and ‘has_many’. The tracking and logging should have as little impact as possible on the application.

> <font color="red">Warning – this table that’s going to grow fast! Make sure to set-up a task to periodically archive and truncate.</font>

##Add the ability to control the level of tracking

We need an application configuration variable that sets the amount of tracking being done. The options are **:none**, **:all** and **:login**. Not setting the variable is the same as **:none**.

Add the following line in each of the three environment config files: development.rb, test.rb, production.rb.

```ruby
# level of Devise usage tracking - :all, :login, :none (default)
config.devise_usage_log_level = :all
```

##And finally – log the tracked data

Write the [DeviseUsageLog.log] method we called in the Devise controllers.

In **model/devise_usage_log.rb**:

```ruby
def self.log(resource, new_action)
    return unless User.valid_user?(resource) \
                  && (Rails.configuration.respond_to? :devise_usage_log_level)

    level = Rails.configuration.devise_usage_log_level
    if level == :all || (level == :login && new_action == DeviseAction::Login)
      resource.log_devise_action(new_action)
    end
  end
```

In **model/user.rb**:

```ruby
def self.valid_user?(resource)
    resource && resource.kind_of?(User) && resource.valid?
  end
 
  def log_devise_action(new_action)
    DeviseUsageLog.create!(user_id: id, role: role, user_ip: current_sign_in_ip, username: username, action: new_action)
  end
```

##Tracking Reports

The DeviseUsageLog model can now be used to list out the Devise activity anyway you want.

One example, is the report I added to the Admin tab of [jehughes/rails4-example](https://github.com/jehughes/rails4-example). The source can be found in the following places:

Controller: [app/controllers/devise_usage_log_controller.rb](https://github.com/jehughes/rails4-example/blob/master/app/controllers/devise_usage_log_controller.rb)
Model: [app/models/devise_usage_log.rb](https://github.com/jehughes/rails4-example/blob/master/app/models/devise_usage_log.rb)
Views: [app/views/devise_usage_log](https://github.com/jehughes/rails4-example/tree/master/app/views/devise_usage_log)

Or a rake task like [lib/tasks/devise_usage.rake](https://github.com/jehughes/rails4-example/blob/master/lib/tasks/devise_usage.rake).