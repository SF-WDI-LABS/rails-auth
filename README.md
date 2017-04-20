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

- Build routes, controllers, and views for user sign up, log in, and log out.
- Implement a `User` model that securely stores passwords with `has_secure_password`.
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

## Authorization

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


### Conclusion

That's it, **authentication** and **authorization** in a nutshell.  Not too bad, right?  


<img src="http://i.giphy.com/TEFplLVRDMWBi.gif" style="max-width: 400px;">



For an example, see <a href="https://github.com/SF-WDI-LABS/rails_blog_app/tree/solution_authorization">a WDI lab solution implementing auth in Rails 4.</a>



### Alternative Strategies

There are a lot of gems that help implement authentication.  For Rails,  <a href="https://github.com/plataformatec/devise">Devise</a> is a very popular gem that will handle your site's authentication for you. [Cancancan](https://github.com/CanCanCommunity/cancancan) is a popular authorization gem. Some "local auth" strategies don't use sessions at all - you could explore [JSON Web Tokens](https://jwt.io/).

Sometimes authenticating with your site isn't enough - sometimes you want data that another organization controls. Here, you're getting into third-party authorization strategies like OAuth (which comes in flavors: 1 or 2). If your Rails app needs data from a third-party API, you could check:  
 - have the API creators provided an official gem for Rails developers to use?  
 - is there an [OmniAuth](https://github.com/omniauth/omniauth) strategy for the API?  
 - do I want to try building more myself with tools like [Oauth2](https://github.com/intridea/oauth2/) gems?
 
<details><summary>**Alternative Strategy Example: OAuth 2 Authorization**</summary>

OAuth 2 is a framework for third-party authorization - allowing one application to access data from another application, on a user's behalf.

In the real world, this might correspond to me walking into a bank and telling the teller you said I could have some of your money every month. We'd hope the teller would be skeptical. Maybe they'd ask that you come into the bank yourself and authorize me to withdraw some specific amount of money on a specific schedule.  They'd probably want a copy of my id so they could verify it was me coming back to withdraw money each time. If everything checked out, the bank would give me a special token or passphrase that I could use to get money now and smooth out the transaction now and for some specified amount of time.

On the web, a classic example is an application that aggregates information stored elsewhere. Say we have a user named Sam. Sam wonders: do the times I commit to github have any relationship to the times I'm most active on facebook?  If Sam wants to use a GitFace app that promises to answer that question, he might have to log into his facebook account and github account so the app can access and process his data.  

Sam may not want to give GitFace his login information for both of those sites, so instead GitFace makes an arrangement with github and facebook. Let's consider facebook. When GitFace needs to access restricted data, it will link Sam to a special authorization page the app has set up with facebook.  This page is entirely controlled by facebook - and it relies on GitFace's facebook app id as well as the specific information GitFace is requesting.  Sam will enter in his information in a form on that page.

GitFace never interacts directly with  Sam's login information.  If Sam is authenticated and agrees to share the resource GitFace needs, facebook's server sends back a response that redirects Sam back to the GitFace app.  The response also includes a special code specific to the data GitFace has requested. 

In the background, GitFace then sends a new request to facebook - it needs to convert Sam's permission code into a token.  In order to get the code converted, GitFace also needs to securely identify itself to facebook by telling facebook its client secret.

If the permission code and client secret check out, facbook issues a token that GitFace can use to access the materials approved by Sam. This usually has some expiration time so that the user doesn't have to re-authenticate on every step.  

</details>


