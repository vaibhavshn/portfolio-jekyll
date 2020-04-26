---
layout: post
title: Voice Controlled IoT Automation (Google Assistant)
category: project
---


Tools/Apps we'll be using

- Google Assistant
- [IFTTT](https://ifttt.com/)
- [Adafruit IO](https://io.adafruit.com/)
- Raspberry Pi (Any Model)
- Sensors of your choice!

I'll just be explaining what you need to do to link the two: the Raspberry Pi to Google Assistant.

What helps us achieve that is IFTTT and Adafruit.

You can use other tools from providers such as AWS, Azure, GCP.
But Adafruit seems beginner friendly, so we'll go with this for now.

## 1. Setting Up Adafruit IO

Adafruit IO is a service that makes activities such as streaming, and logging data from or to IoT devices from web apps.
It works with almost every IoT device.

We'll try it on a Raspberry Pi 3.

- Create an Adafruit account from https://io.adafruit.com
- Go to https://io.adafruit.com
- Create a new dashboard, name it anything you like, say "Home Automation".
- Click on your dashboard, click the `+` icon for `Create a new block`.
  A block can be anything related to a form of data, say a button, slider, image, indicator etc.
- Create a simple `toggle` button block. name it `light` or anything you like. 
  Leave everything else default, and note the ON and OFF keywords, these will be used to know the current state of the toggle. 
  However, it is best to change them to `on` and `off`, since the assistant sends it in lowercase, but it wouldn't matter because this data will be sent to the Raspberry Pi.
- Now, you can toggle it ON or OFF from the dashboard itself.
- Adafruit feed is now set up!

> If you click the block name, you can see logs of data changes with a nice graph.



## 2. Setting up IFTTT

You'll need to have Google Assistant enabled on your phone.

- Download the IFTTT app on your phone.
- Create an account and log in.
- Click `Get More` on the home screen of the app.
- Click `Make your own Applets from scratch +` at the top below the search bar.
- Click `This` and select Google Assistant by searching for it.
- For simplicity, let's just make it understand a phrase with a `text ingredient`.
  Here, the ingredient will be the state of the toggle.
- On clicking the said option, enter the following in the `What do you want to say?` field.
  ```
  Switch $ light
  ```
  Here, whenever you say something like `Turn on the light` or `Turn off the light`, the respective word will be sent to the field, which we will be using to toggle the light in Adafruit.
- Add any response you like the Assistant to have such as "The light is turned $", where `$` will be replaced by the said term.
- Select language `English `and click Continue.
- Click `That` and search for Adafruit. 
  You'll have to log in to the Adafruit account from the app to link it to IFTTT.
- Click `Send data to Adafruit IO`.
- Select the feed name from drop down, here `light`.
  In the `Data to save` field, click `Insert ingredients` and select `TextField`.
- Click Continue and then Finish.


We have set up everything we've needed except Raspberry Pi.


## 3. Setting up Adafruit library on a Raspberry Pi

- Install Adafruit IO Python package

  ```
  pip3 install adafruit-io
  ```
- Here is an example code to subscribe to a feed via MQTT from their [repository](https://github.com/adafruit/Adafruit_IO_Python).
  ```python
  # Example of using the MQTT client class to subscribe to a feed and print out
  # any changes made to the feed.  Edit the variables below to configure the key,
  # username, and feed to subscribe to for changes.
  
  # Import standard python modules.
  import sys
  
  # Import Adafruit IO MQTT client.
  from Adafruit_IO import MQTTClient
  
  # Set to your Adafruit IO key.
  # Remember, your key is a secret,
  # so make sure not to publish it when you publish this code!
  ADAFRUIT_IO_KEY = 'YOUR_AIO_KEY'
  
  # Set to your Adafruit IO username.
  # (go to https://accounts.adafruit.com to find your username)
  ADAFRUIT_IO_USERNAME = 'YOUR_AIO_USERNAME'
  
  # Set to the ID of the feed to subscribe to for updates.
  FEED_ID = 'DemoFeed'
  
  
  # Define callback functions which will be called when certain events happen.
  def connected(client):
      # Connected function will be called when the client is connected to Adafruit IO.
      # This is a good place to subscribe to feed changes.  The client parameter
      # passed to this function is the Adafruit IO MQTT client so you can make
      # calls against it easily.
      print('Connected to Adafruit IO!  Listening for {0} changes...'.format(FEED_ID))
      # Subscribe to changes on a feed named DemoFeed.
      client.subscribe(FEED_ID)
  
  def subscribe(client, userdata, mid, granted_qos):
      # This method is called when the client subscribes to a new feed.
      print('Subscribed to {0} with QoS {1}'.format(FEED_ID, granted_qos[0]))
  
  def disconnected(client):
      # Disconnected function will be called when the client disconnects.
      print('Disconnected from Adafruit IO!')
      sys.exit(1)
  
  def message(client, feed_id, payload):
      # Message function will be called when a subscribed feed has a new value.
      # The feed_id parameter identifies the feed, and the payload parameter has
      # the new value.
      print('Feed {0} received new value: {1}'.format(feed_id, payload))
  
  
  # Create an MQTT client instance.
  client = MQTTClient(ADAFRUIT_IO_USERNAME, ADAFRUIT_IO_KEY)
  
  # Setup the callback functions defined above.
  client.on_connect    = connected
  client.on_disconnect = disconnected
  client.on_message    = message
  client.on_subscribe  = subscribe
  
  # Connect to the Adafruit IO server.
  client.connect()
  
  # Start a message loop that blocks forever waiting for MQTT messages to be
  # received.  Note there are other options for running the event loop like doing
  # so in a background thread--see the mqtt_client.py example to learn more.
  client.loop_blocking()
  ```
- Insert the following data into the code above:

  - Adafruit IO API Key
  - Adafruit IO Username
  - Feed name, here `light`.
- Note the subscribe function, it is the function that is run on receiving data on a feed.
- Modify the function to suit your need.
- An example:

  ```python
  # Imports
  from RPi import GPIO
  
  # GPIO Setup code
  
  def message(client, feed_id, payload):
      # Message function will be called when a subscribed feed has a new value.
      # The feed_id parameter identifies the feed, and the payload parameter has
      # the new value.
      print('Feed {0} received new value: {1}'.format(feed_id, payload))
  	if payload == "on":
          payload = True
      else:
          payload = False
  	GPIO.output(pin, payload)
  ```
- That's it! Now, run the code on your Raspberry Pi, make sure it is connected to the internet and connect an LED or any sensor or device of your choice.



Now try telling the Assistant to turn it on! This was a simple example, once you understand the basics, you can extend it even more according to your needs!





