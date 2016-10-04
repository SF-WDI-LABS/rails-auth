<!--
Creator: Team
Last Edited by: Brianna
Location: SF
-->

![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)

# Auth in Rails

### Why is this important?
<!-- framing the "why" in big-picture/real world examples -->
*This workshop is important because:*

Many apps track user data and allow users to sign up, log in, and log out. Today we'll look at a strategy to implement these features.

### What are the objectives?
<!-- specific/measurable goal for students to achieve -->
*After this workshop, developers will be able to:*

- Implement a `User` model that securely stores passwords with `has_secure_password`.
- Build routes, controllers, and for user sign up, log in, and log out.
- Differentiate authentication and authorization.

### Where should we be now?
<!-- call out the skills that are prerequisites -->
*Before this workshop, developers should already be able to:*

- Describe the role of routes, controllers, views, and models.   

### Aside: Devise

Some of you have heard of <a href="https://github.com/plataformatec/devise">Devise</a>, a gem that will handle auth for you. That's cool and all, but they recommend that if you're new to rails:

> If you are building your first Rails application, we recommend you do not use Devise. Devise requires a good understanding of the Rails Framework. In such cases, we advise you to start a simple authentication system from scratch. ... <a href="https://github.com/plataformatec/devise#starting-with-rails"\>[docs]</a>

**That's what we're going to do!**

<img src="https://cloud.githubusercontent.com/assets/1329385/11758689/15df1a8c-a023-11e5-9e59-065e5bb5dd23.gif">

### What is "auth"?

People generally mean two different things when they say "auth":

1. **Authentication** - confirming that a user is who they say they are.
2. **Authorization** - determining whether a user is permitted to perform an action.


### Auth Strategy: Sessions

HTTP is designed to be stateless.  We could have users submit login details for every request, but that would get really tedious really quickly.  

We'll store information about the logged in user in a **session**.  (See Rails Security Guide [Sessions section](http://guides.rubyonrails.org/security.html#session) for a refresher.)

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

### Handling Passwords

We'll use the gem `bcrypt` so that we **never store user passwords as plain text**.  BCrypt is a hashing algorithm, and people like it because you can customize how secure a job it does based on a "cost" factor.  The [bcrypt gem](https://github.com/codahale/bcrypt-ruby) will let us simply and easily hash and salt our passwords to obscure them with the BCrypt algorithm. Let's try it out.

#### Independent Practice: What does `bcrypt` do?

1. In your Terminal, install the bcrypt gem: `gem install bcrypt`.

1. Start up `irb` or `pry`, and try out the console commands below:

  ```bash
    2.3.0 :001 > require 'bcrypt'
     => true
    2.3.0 :002 > my_password = BCrypt::Password.create('secret')
     => "$2a$10$7z71iDWik1luWbDR.2uC2ObB5fPT5jSbWFfsp3YGnsZrcRIHIkjne"
    2.3.0 :003 > my_password == "secret"
     => true
    2.3.0 :004 > my_password == "oops"
     => false
  ```

1. Play around with different values.  What happens if you run `"secret"`  through the `bcrypt` gem again? Do you get the same output?


#### `has_secure_password`

In Rails, the `bcrypt` gem gives us an intensely easy-to-use `has_secure_password` method that we can add to a class we want to authenticate with.

```rb
class User < ActiveRecord::Base
  # first make sure bcrypt is included in Gemfile
  has_secure_password

end
```

This will make sure the passwords are NOT stored as plain text in the database.  It will store the password in the database under the column `password_digest`.

It also adds an `authenticate` method which can check a given password against the hashed and salted version in the database:

```rb
  # some controller somewhere
  user = User.find_by_email(params[:email])
  if user && user.authenticate(params[:password])
    # ...
```

Note that we *could* use `bcrypt` to write these methods ourselves - in fact, that's exactly what the source code does!  See `has_secure_password`'s [documentation](http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html) and [source code](https://github.com/rails/rails/blob/5-0-stable/activemodel/lib/active_model/secure_password.rb#L53).


### Session Store for User ID

Whenever we authenticate a user, we will set the session hash's `user_id` to the `user.id`:

```rb
session[:user_id] = user.id
```

You'll want to do this at **signup** and **log in**.

**Check for understanding**: How would we use the session hash to log a user *out*?

## Conclusion

That's it, **authentication** in a nutshell.  Not too bad right?  


<img src="http://i.giphy.com/TEFplLVRDMWBi.gif" style="max-width: 400px;">

---

You've seen **authentication**, but for many apps you'll still want to add **authorization**.

* We suggest making a `current_user` method in your `ApplicationController` or in a helper file so that each controller doesn't need to implement a way to check which user is logged in.

```rb
  def current_user
    @current_user ||= User.find(session[:user_id]) if session[:user_id]
  end
  helper_method :current_user
```

* Then, in each of your controllers, check whether the `current_user` should be allowed to do the action they're trying to. You can use `before_action` to keep this code DRYer.

* Consider adding conditionals to your views. For example, don't show links to users who can't use them.
