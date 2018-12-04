# Sending Emails on AWS Hosted Servers

This guide will walk you through setting up SMTP on AWS for our EC2 hosted systems.
Generally we want to do do this so that application notifications will be sent by email.
Some systems, e.g., spotlight, require email to be set up, and you cannot build
an exhibit unless the server can send email. For other systems, e.g., Hyrax,
there might also be application level setup necessary to send notifications by email.
See [this samvera documentation](http://samvera.github.io/email_notifications.html) for details on how to make Hyrax send notifications by email.

### Getting AWS SMTP credentials
1. Go to [Amazon's Simple Email Service](https://console.aws.amazon.com/ses/home?region=us-east-1).
2. Choose SMTP Settings from the menu on the left, and click "Create my SMTP credentials"
3. Set a username specific to the application you're setting up.
4. Screenshot the SMTP credentials on the screen. Make sure to record them because this is the only time the system will display them. You can also download a credentials file.

### Validate the email address you'll use
You will only be able to send email to specific validated email addresses. If you try to send to an unvalidated email address you'll get an error like:
```
Net::SMTPFatalError: 554 Message rejected: Email address is not verified.
The following identities failed the check in region US-EAST-1:
youremail@curationexperts.com
```
1. Go to the `Email Addresses` link in the left hand menu of the AWS `Simple Email Service`.
1. Click "Verify a new email address" and enter the email you'd like to use. It must be an email you have access to.
1. Go check your mail and click the "verify" link when the confirmation email arrives.

### Set up SMTP for your application
1. Add a block like this to `config/environments/production.rb`:
```
config.action_mailer.perform_deliveries = true
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
  address: 'email-smtp.us-east-1.amazonaws.com',
  port: '587',
  user_name: 'MY_USER_NAME',
  password: 'MY_PASSWORD',
  enable_starttls_auto: true
}
```
2. And a file `app/mailers/test_mailer.rb`:
```
  class TestMailer < ApplicationMailer
    def test_email
      mail(
        from: "you@yourdomain.com",
        to: "you@yourdomain.com",
        subject: "Test mail",
        body: "Test mail body"
      )
    end
  end
```
On a rails console, you should be able to run `TestMailer.test_email.deliver` and see your email delivered.
Assuming it works as expected you should see something like:

```
irb(main):002:0> TestMailer.test_email.deliver
TestMailer#test_email: processed outbound mail in 0.6ms
Sent mail to bess@curationexperts.com (1391.8ms)
Date: Tue, 04 Dec 2018 13:58:23 -0500
From: bess@curationexperts.com
To: bess@curationexperts.com
Message-ID: <5c06ce4f5f648_11221044f605862c@geniza-cd.mail>
Subject: Test mail
Mime-Version: 1.0
Content-Type: text/plain;
 charset=UTF-8
Content-Transfer-Encoding: 7bit

Test mail body
=> #<Mail::Message:68728440, Multipart: false, Headers: <Date: Tue, 04 Dec 2018 13:58:23 -0500>, <From: bess@curationexperts.com>, <To: bess@curationexperts.com>, <Message-ID: <5c06ce4f5f648_11221044f605862c@geniza-cd.mail>>, <Subject: Test mail>, <Mime-Version: 1.0>, <Content-Type: text/plain>, <Content-Transfer-Encoding: 7bit>>
```
