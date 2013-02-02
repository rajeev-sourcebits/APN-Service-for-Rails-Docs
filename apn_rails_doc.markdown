   	 	
# Implementing Apple Push Notification Service in Rails #

## Push Notifications Overview ##

Getting push to work for your app takes quite a bit of effort. This is a puzzle with many pieces. Here is an overview:

![](http://cdn3.raywenderlich.com/wp-content/uploads/2011/05/Push-Overview.jpg)

1. An app enables push notifications. The user has to confirm that he wishes to receive these notifications.
2. The app receives a “device token”. You can think of the device token as the address that push notifications will be sent to.
3. The app sends the device token to your server.
4. When something of interest to your app happens, the server sends a push notification to the Apple Push Notification Service, or APNS for short.
5. APNS sends the push notification to the user’s device.


## Converting and Adding Certificate ##

Once you have the certificate from Apple for your application, export your key
and the apple certificate as p12 files. Here is a quick walkthrough on how to do this:

1. Click the disclosure arrow next to your certificate in Keychain Access and select the certificate and the key.
2. Right click and choose `Export 2 items.
3. Choose the p12 format from the drop down and name it `cert.p12`.
Now convert the p12 file to a pem file:

`$ openssl pkcs12 -in cert.p12 -out apple_push_notification_production.pem -nodes -clcerts`

If you are using a development certificate, then change the name to apple_push_notification_development.pem instead.
Add the .pem file in config/ directory of the Rails application.

## Installing Gem for Push Notifications 


Gem to use for Apple push notifications is “apn_on_rails” [Github Link] (https://github.com/PRX/apn_on_rails)

To add the gem to your Rails app just add this line to your Gemfile.

`'gem apn_on_rails'`

or Type in this command at terminal

`$ sudo gem install apn_on_rails`


## Registering your App for Apple Push Notifications ##

Before adding push notification service to your Rails app, you have to register your app for the Push Notification Service with the certificate created earlier.

### Register your App with credentials as (Ruby Code) ###

```ruby
app = APN::App.new
app.apn_dev_cert = File.read('config/apple_push_notification_development.pem')
app.save
```
The code creates a new APN::App, adds a  Developer certificate and saves it to the database.

## Registering the devices of the users to your Rails Application ## 

After registering the app for push notifications, The next step is to register the devices of all the users of your Application. Registering the user devices to the service is through there “device tokens”. When the user registers to the app in the “client side”, the device token is retrieved and sent to the “Rails App” in the POST request.

Use that device tokens to register the users for the Apple Push Notification Service.

### Register the users to the service as (Ruby code) ###

```ruby
device = APN::Device.new
device.token = self.device_token
device.app_id=app.id
device.save
```
The code creates a new App Device Instance.
Adds the device token of the user fetched from the client app.
Adds an app_id to the device with the id of your App created earlier and then save the device to the database.
Now your user is ready to accept the Push Notification from the server.

## Single Notifications  ##

If you want to send individual notifications.

```ruby
notification = APN::Notification.new
notification.device = device
notification.badge = 5
notification.sound = true
notification.alert = "Message by Admin"
notification.save
```

The code creates a Notification Instance.
Adds a device you want to send notifications too.
Adds badge, sound and alert message to the notification and save it to the database.

Then the send the notification to the device using

```ruby
APN::App.send_notifications
```

## Group Notifications ##

The gem does has support for group notificatiosn too. Sending messages to many devices at the same time.

### Get all the registered devices using ###

```ruby
devices = APN::Device.all
```

### Get the App registered for the service ###

```ruby
app = APN::App.first
```

For sending group notifications,we have to create a group and add the devices array to the group.

### Create a group and add name and app_id and save it to the database ###

```ruby
  group = APN::Group.new
  group.name = 'products'
  group.app_id = app.id
  group.save
```

### Adding the devices array to the group ###

```ruby
group.devices = devices
```

### Adding a Group Notification 

Create a Group Notification instance. Add the group instance to the notification group. Add alert, badge messages and save it to the database

```ruby
notification = APN::GroupNotification.new
notification.group = group
notification.alert = 'Event got locked'
notification.badge = 4
notification.sound = true
notification.save
```

Then send the Group notification to all the devices as

```ruby
APN::App.send_group_notifications
APN::App.process_devices
```

This sends the message to all the devices registered.








