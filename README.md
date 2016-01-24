# Authentication with Devise

## Learning Objectives

  1. Describe the major architecture and modules of Devise.

## Overview

[Devise] is a gem for when you have a lot of needs.

Want to send confirmation emails, lock user accounts after a certain number of failed login attempts, and send password resets? Devise can do that.

Want to allow multiple models to be signed in, each with different roles, granting different permissions? Devise can handle that too.

Devise has an extensive collection of generators. It includes standard views and controllers. It will generate templates that include Bootstrap for you.

It will also be, at times, a giant pain, because no magic is without a price. Devise abstracts away a lot of implementation details. That can be very nice, but it can also make it challenging to setup and debug when things don't go exactly to plan.

## Architecture

Devise is a Rails [engine]. That means it's basically a Rails app that sits inside your Rails app. It has its own views and controllers, and defines its own routes.

It does not define models for you, but it does have generators that make the process of creating a Devise-compliant User model very easy.

Devise is made up of modules. Modules are applied to your `User` model, so you should read these as abilities that User accounts can have.

       * Database Authenticatable: encrypts and stores a password in the database to validate the authenticity of a user while signing in. The authentication can be done both through POST requests or HTTP Basic Authentication.
       * Omniauthable: adds OmniAuth (https://github.com/intridea/omniauth) support.
       * Confirmable: sends emails with confirmation instructions and verifies whether an account is already confirmed during sign in.
       * Recoverable: resets the user password and sends reset instructions.
       * Registerable: handles signing up users through a registration process, also allowing them to edit and destroy their account.
       * Rememberable: manages generating and clearing a token for remembering the user from a saved cookie.
       * Trackable: tracks sign in count, timestamps and IP address.
       * Timeoutable: expires sessions that have not been active in a specified period of time.
       * Validatable: provides validations of email and password. It's optional and can be customized, so you're able to define your own validations.
       * Lockable: locks an account after a specified number of failed sign-in attempts. Can unlock via email or after a specified time period.

That's from the [Devise docs][Devise].

## Setup

Several of these modules come added for you by default if you run

    `rails generate devise User`

This generates a `User` model which includes this helpful bit at the top:

    # Include default devise modules. Others available are:
    # :confirmable, :lockable, :timeoutable and :omniauthable
    devise :database_authenticatable, :registerable,
           :recoverable, :rememberable, :trackable, :validatable,

That's Devise wiring itself into the model. It also creates a migration for all the fields it needs.

Let's look at the highlights.

### [database_authenticable]

This adds a `valid_password?(password)` method to the model. The password is stored securely in the database.

### [registerable]

Registerable gives you `User.new_with_session(params, session)`, which lets you initialize a `User` from session data (like a token from Facebook) in addition to params.

### [recoverable]

Recoverable gives you password resets, like so:

    # resets the user password and save the record, true if valid passwords are given, otherwise false
    User.find(1).reset_password('password123', 'password123')

    # only resets the user password, without saving the record
    user = User.find(1)
    user.reset_password('password123', 'password123')

    # creates a new token and send it with instructions about how to reset the password
    # (this one requires a mailer.)
    User.find(1).send_reset_password_instructions

### [rememberable]

This lets you remember a user and associate them with a `User` object in the database without them having to log in. It works by storing a token in cookies.

    User.find(1).remember_me!  # regenerating the token
    User.find(1).forget_me!    # clearing the token

### [trackable]

Track information about your user sign in. It tracks the following columns:

 * sign_in_count - Increased every time a sign in is made (by form, openid, oauth)
 * current_sign_in_at - A timestamp updated when the user signs in
 * last_sign_in_at - Holds the timestamp of the previous sign in
 * current_sign_in_ip - The remote ip updated when the user sign in
 * last_sign_in_ip - Holds the remote ip of the previous sign in

### [validatable]

The documentation on this is quite clear:

    Validatable creates all needed validations for a user email and password. It's optional, given you may want to create the validations by yourself. Automatically validate if the email is present, unique and its format is valid. Also tests presence of password, confirmation and length.

### [lockable]

Handles blocking a user's access after a certain number of attempts. Lockable accepts two different strategies to unlock a user after it's blocked: email and time. The former will send an email to the user when the lock happens, containing a link to unlock its account. The second will unlock the user automatically after some configured time (ie 2.hours). It's also possible to setup lockable to use both email and time strategies.


### [omniauthable]

Honestly, this one doesn't give you a whole lot more than omniauth already does. It does set some (but not all!) of the routes for you. That's a nice touch.


## Take a breath

Devise is rather a lot to take in all at once. The thing about large and magical frameworks like Devise (and Rails for that matter) is that they aren't hard in the way that math problems are hard, but they are hard in the way that your taxes are hard. When I'm using frameworks like this, I spend an awful lot of time alt-tabbing over to Chrome to figure out what pieces I need and how to use them. If you find yourself doing that too, worry not. That's how everyone programs.


[Devise]: https://github.com/plataformatec/devise
[engine]: http://guides.rubyonrails.org/engines.html
[registerable]: http://www.rubydoc.info/github/plataformatec/devise/master/Devise/Models/Registerable
[database_authenticable]: http://www.rubydoc.info/github/plataformatec/devise/master/Devise/Models/DatabaseAuthenticable
[recoverable]: http://www.rubydoc.info/github/plataformatec/devise/master/Devise/Models/Recoverable
[trackable]: http://www.rubydoc.info/github/plataformatec/devise/master/Devise/Models/Trackable
[validatable]: http://www.rubydoc.info/github/plataformatec/devise/master/Devise/Models/Validatable
[omniauthable]: http://www.rubydoc.info/github/plataformatec/devise/master/Devise/Models/Omniauthable