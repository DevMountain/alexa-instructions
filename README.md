# Building an Alexa Skill

With the Alexa Skills kit you can make a program that can respond to requests from an Alexa device. Here, you'll learn how to build an app using Node that can handle these requests. Then, you'll learn how to create an AWS Lambda function to host your skill and get it connected to your Alexa.

## Step 1\. Writing the code for your Alexa Skill

We're going to be using Amazon's [alexa-sdk](https://www.npmjs.com/package/alexa-sdk) for Node to write our application.

1. Set up a new project. Make a new folder called HelloAlexa.

1. Go into your new directory and run `npm init -y` in the terminal. Then run `npm install --save alexa-sdk` to save the Alexa SDK as a dependency.

1. Now that you've set up your dependencies, create an `index.js` file and open it up.

1. Start out in `index.js` by importing the Alexa package we just downloaded.

  `var Alexa = require('alexa-sdk')`

1. The first part of the application we'll want to implement are the event handlers. We defined the events our app will handle in the JSON file we just filled out in the developer portal. The handler object we create here will contain a key for each event we defined earlier. Here is an example that would work with the demo JSON file from the previous section:

    ```
    var handlers = {
         'HelloWorldIntent': function(){
             this.emit(':tell', 'Hello!');
         },
         'GetWeatherIntent': function(){
             var temp;

             //gets location from the slot we defined earlier
             var location = this.event.request.intent.slots.Location.value;

             //then make your http request using the information from above
             request('weatherAPIUrl').then(function(response){
                    response.data.temperature = temp;
                    this.emit(':tell', 'The temperature is ' + temp);
             });
         }
     }
    ```

    In order to have Alexa report back to the user, you use the `this.emit` function. The first parameter passed in (in this case, we used `':tell'`), tells the service how to respond to the user. If you use `':tell'`, Alexa will speak the text you pass in. If you pass in `':tellWithCard'` a card will show up in the Alexa app with the informtion you pass in.

1. Next, we need to register the `handlers` object we created above with the Alexa SDK.

  ```
  exports.handler = function(event, context, callback){
      var alexa = Alexa.handler(event, context);
      alexa.registerHandlers(handlers);
      alexa.execute();
  }
  ```

  This code is tells the application what handlers have been defined and allows requests our app receives to be routed to the correct handler. In the last line, we tell the app to actually run with `alexa.execute()`.
