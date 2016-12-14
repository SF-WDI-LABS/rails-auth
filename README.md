# <img src="https://cloud.githubusercontent.com/assets/7833470/10899314/63829980-8188-11e5-8cdd-4ded5bcb6e36.png" height="60"> Rails Auth

| Objectives |
| :--- |
| Implement a User model that securely stores passwords. |
| Build routes, controllers, and views necessary for a typical user flow. |
| Add a `current_user` application-level helper method to keep track of the currently signed in user. |

## Devise

<img src="https://cloud.githubusercontent.com/assets/1329385/11758689/15df1a8c-a023-11e5-9e59-065e5bb5dd23.gif">

I get it, some of you have heard of <a href="https://github.com/plataformatec/devise" target="_blank">Devise</a>. That's cool and all but they recommend that if you're new to rails:

> If you are building your first Rails application, we recommend you do not use Devise. Devise requires a good understanding of the Rails Framework. In such cases, we advise you to start a simple authentication system from scratch. ... <a href="https://github.com/plataformatec/devise#starting-with-rails" target="_blank">[docs]</a>

## Auth in Rails

In order to "roll our own" authentication we're going to follow <a href="https://gist.github.com/eerwitt/b36db29a025366037925" target="_blank">this tutorial</a>.

Don't copy/paste the tutorial, this is the most important part and to avoid copy-pasta we're going to change some core pieces.

## Requirements

1. Your rails app can't be named "gif_vault".
1. Your rails app must not include any routes related to "gifs".
1. You must be able to explain these terms:
  * <details>
      <summary>`bcrypt`</summary>

      > The bcrypt function is the default password hash algorithm for BSD and other systems ... <a href="https://en.wikipedia.org/wiki/Bcrypt" target="_blank">[wiki]</a>

      A method of doing one-way hashes of passwords.
    </details>
  * <details>
      <summary>CSRF Token</summary>

      > Synchronizer token pattern is a technique where a token, secret and unique value for each request, is embedded by the web application in all HTML forms and verified on the server side. The token may be generated by any method that ensures unpredictability and uniqueness ... <a href="https://en.wikipedia.org/wiki/Cross-site_request_forgery#Prevention" target="_blank">[wiki]</a>

    </details>
  * <details>
    <summary>`has_secure_password`</summary>
    This adds methods to set and authenticate against a BCrypt password. This mechanism requires you to have a password_digest attribute.
    <a href="http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html">[docs]</a>
   </details>

## Challenges

1. Don't let an authenticated user signup or login a second time.
1. Implement `password_confirmation` for new users.
1. Create the ability to "<a href="http://railscasts.com/episodes/274-remember-me-reset-password" target="_blank">remember me</a>" for users logging in.
  * How long do you remember them for?
1. Add ability to <a href="http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html#method-i-has_secure_password" target="_blank">reset a password</a>.
1. Instead of a separate login view, give your site a navbar and add the login form to the nav.
1. Support <a href="https://github.com/intridea/omniauth-github" target="_blank">3rd-party authentication</a> on your own.
  * Could be github auth.

## Docs & Resources

* <a href="https://github.com/SF-WDI-LABS/shared_modules/tree/master/04-ruby-rails/cookies-and-sessions/28">Cookies & Sessions</a>
* <a href="http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html#method-i-has_secure_password" target="_blank">`has_secure_password`</a>
* <a href="https://www.railstutorial.org/book/modeling_users" target="_blank">Rails Tutorial on Modeling Users</a>
* <a href="http://railscasts.com/episodes/250-authentication-from-scratch" target="_blank">Kinda Old Railscast</a>
* <a href="https://www.codecademy.com/en/learn/rails-auth" target="_blank">Code Academy: Auth</a>
