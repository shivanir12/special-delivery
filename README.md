# Special Delivery

__A Mailgun Webhook Event Manager__

Special Delivery is a Rails engine that enables powerful and easy-to-use callbacks in response to Mailgun events POSTed via webhook.

Say what now? Example time:

Let's say you have an application that sends an email to notify a user that they have won the lottery. This email is obviously pretty important, so we'd like to send a message to an admin if the lotter-winner email bounces. How the heck do we do that?

Special Delivery saves a reference to each outgoing email messages to recreate state when Mailgun informs your application that a particular event occured for that message.

So by using Special Delivery when sending out a lottery-winner email, the application is then able to listen for bounce events for that particular email, and evaluate a callback with knowlege of the user record to which the email was originally sent.


## Use
### Routes File
Mount Special Delivery as an engine within your `config/routes.rb` at a path of your choosing.

```ruby
mount SpecialDelivery::Engine => "/email_events"
```

### Your Custom Callback Class File
Create a new class to handle callbacks initiated by Mailgun POST requests made to your app.  Callbacks for the specific types of Mailgun events should be defined as class methods on your class. Your class can respond to as many or as few of Mailgun's event types as you would like. Event types include `bounce`, `click`, `complaint`, `delivery`, `drop`, `open`, and `unsubscribe`.

So if we want a callback class to manage sending a message to an admin when our lotter winner emails bounce, we would write:

```ruby
class LotterEmailCallbacks::Winner
  def self.bounce(user)
  	send_message_to_admin "#{user.name} did not receive lottery winner email."
  end
end
```

### Mailer Files
Include the Mailer helper module in your mailer class (e.g. `app/mailers/lotter_mailer.rb` if you wanted to use Special Delivery in your LotterMailer.

```ruby
class LotteryMailer < ActionMailer::Base
  include SpecialDelivery::Mailer
  
  ...
```

Within your mailer's methods, pass your `mail` method calls into `special_delivery` as a block.

```ruby
def winner_email(user)
  special_delivery(
    :callback_class  => LotterEmailCallbacks::Winner,
    :callback_record => user
  ) do
  	mail(:to => user.email, :subject => 'All the Monies!')
  end
end
```