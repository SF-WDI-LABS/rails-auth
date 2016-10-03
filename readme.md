# Auth in Rails

## Overview

**Auth** entails two _authy_ components:

1. **Authentication** - identifying a user.
2. **Authorization** - determining whether a user is permitted to perform an action.


Because HTTP is designed to be stateless, and we don't want to require users to submit login details for every operation. We'll use **session** information in Rails to maintain the state of "who" the current user is.


Below is a _summary_ of what we will use:

### Routes

```rb
  get '/login', to: 'sessions#new'
  post '/login', to: 'sessions#create'
  get '/logout', to: 'sessions#destroy'

  get '/signup', to: 'users#new'
  post '/users', to: 'users#create'
```

### Models

We'll need one model: `Users`.  We'll add `name`, `email` and `password_digest` attributes.

```
# Table name: users
#
#  id              :integer          not null, primary key
#  name            :string
#  email           :string
#  password_digest :string
```

### Handling passwords

We'll use the gem `bcrypt` which when enabled in Rails gives us the [`has_secure_password`](https://github.com/rails/rails/blob/master/activemodel/lib/active_model/secure_password.rb#L36) method we can call from our `Users` Class.

```rb
class User < ActiveRecord::Base
  # first add bcrypt to Gemfile
  has_secure_password

end
```

This will make sure the passwords are NOT stored as plain text in the DB.  It will store the password in the DB under the column `password_digest`.

It also adds an `authenticate` method which can check a given password against the hashed and salted version in the database.

```rb
  # some controller somewhere
  user = User.find_by_email(params[:email])
  if user && user.authenticate(params[:password])
   ...
```

## Tracking the user ID

Whenever we authenticate a user, we will set the session hash's `user_id` to the `user.id`:

```rb
session[:user_id] = user.id
```

You'll want to do this at **signup** and **log in**.

## Conclusion

That's it, **authentication** in a nutshell.  Not too bad right?  


<img src="http://i.giphy.com/TEFplLVRDMWBi.gif" style="max-width: 400px;">

---

You're not quite done yet though.  You have **authentication**, but you'll still want to:



* Add a `current_user` method in your `ApplicationController` or in a helper file so that each controller doesn't need to implement a way to check users for **authorization**.

```rb
  def current_user
    @current_user ||= User.find(session[:user_id]) if session[:user_id]
  end
  helper_method :current_user
```
* Add **authorization** -  go to each of your controllers and check whether the `current_user` should be allowed to do the action they're trying to.
* Consider adding conditionals to your views - don't show links to users that can't use them.



## Resources

* `has_secure_password` in Rails - https://github.com/rails/rails/blob/master/activemodel/lib/active_model/secure_password.rb#L36
