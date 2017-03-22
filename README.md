# Building an Alexa Skill

With the Alexa Skills kit you can make a program that can respond to requests from an Alexa device. Here, you'll learn how to build an app using Node that can handle these requests. Then, you'll learn how to create an AWS Lambda function to host your skill and get it connected to your Alexa.

## Step 1\. Writing the code for your Alexa Skill

We're going to be using Amazon's [alexa-sdk](https://www.npmjs.com/package/alexa-sdk) for Node to write our application.

1. Set up a new project. Make a new folder called HelloAlexa.

1. Go into your new directory and run `npm init -y` in the terminal. Then run `npm install --save alexa-sdk` to save the Alexa SDK as a dependency.

1. Now that you've set up your dependencies, create an `index.js` file and open it up.

1. Start out in `index.js` by importing the Alexa package we just downloaded.

  `var Alexa = require('alexa-sdk')`

1. First, we're going to create the Intent Schema for our action. The Intent Schema is a JSON object that tells the Alexa service the actions our application will handle. When you set up your Alexa skill on the Alexa Developer Portal, you'll need to have your schema ready. For now, just save it in a .js file inside your project. Here is the Intent Schema for the skill we're going to make in this project:

    ```
    {
      "intents": [
        {
            "intent": "HelloWorldIntent"
        },
        {
            "intent": "GetWeatherIntent",
            "slots": [
                {
                    "name": "city",
                    "type": "AMAZON.US_CITY"
                }
            ]
        }
      ]
    }
    ```
This defines a skill that has two intent, a HelloWorldIntent, and a GetWeatherIntent. The GetWeatherIntent has an additional section called slots, which tells us that this intent will require information from the user (in this case a city) in order to run. In order to define a slot, you must give it a name and a type. Amazon has [a ton of premade types](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/built-in-intent-ref/slot-type-reference), so its best to select one from that list when defining your slots.

1. You'll also need to define a list of Sample Utterances that will be used to invoke your program. Here is a list that would be applicable to the intent schema shown above.

    ```
        HelloWorldIntent say hello
        HelloWorldIntent tell me hello
        GetWeatherIntent get me the weather in {london | city}
        GetWeatherIntent how is the weather in {los angeles | city}
        GetWeatherIntent whats the weather like in {provo | city}
        GetWeatherIntent what is the temperature in {new york | city} today
        GetWeatherIntent how is the weather today in {phoenix | city}
    ```
The more examples you provide, the more accurate Alexa will be at launching your skill at the correct times. For the intent that has a slot, we need to provide an example of something the user might say, along with the name of the slot in the format shown above. Save this as a separate file somewhere in your application. You'll need it later when you sit down with your mentor to set up the alexa skill on the developer portal.

1. The first part of the application we'll want to implement are the event handlers. We defined the events our app will handle in the JSON file we just filled out in the developer portal. The handler object we create here will contain a key for each event we defined earlier. Here is an example that would work with the demo JSON file from the previous section:

    ```
    var handlers = {
         'HelloWorldIntent': function(){
             this.emit(':tell', 'Hello!');
         },
         'GetWeatherIntent': function(){
             var temp;

             //gets location from the slot we defined earlier
             var location = this.event.request.intent.slots.City.value;

             //try console logging this.event.request to see what other info you get with each request
             //console.log(this.event.request);

             //then make your http request using the information from above
             request('weatherAPIUrl').then(function(response){
                    response.data.temperature = temp;
                    this.emit(':tell', 'The temperature is ' + temp);
             });
         }
     }
    ```

    In order to have Alexa report back to the user, you use the `this.emit` function. The first parameter passed in (in this case, we used `':tell'`), tells the service how to respond to the user. If you use `':tell'`, Alexa will speak the text you pass in. If you pass in `':tellWithCard'` a card will show up in the Alexa app with the information you pass in.

1. Next, we need to register the `handlers` object we created above with the Alexa SDK.

  ```
  exports.handler = function(event, context, callback){
      var alexa = Alexa.handler(event, context);
      alexa.registerHandlers(handlers);
      alexa.execute();
  }
  ```

  This code is tells the application what handlers have been defined and allows requests our app receives to be routed to the correct handler. In the last line, we tell the app to actually run with `alexa.execute()`.

1. Now the app is ready to be uploaded to AWS. In the next step, we'll go over creating a function in AWS Lambda. But first, find your project folder on your computer and open it. Then compress everything inside the folder to a .zip file. It's extremely important that you open the file and the compress the contents, rather than just compressing the HelloAlexa folder, as that is the only way it will work.

## Step 2. Setting up AWS Lambda

1. Go to [AWS]('https://aws.amazon.com') and sign up for a free account if you don't already have one. The free tier includes 1 million requests per month, so that should be plenty.

1. When making your account select 'N. Virigina' as your data center. This is the only data center that currently supports Alexa requests. If you already have an account, log in and change your data center to N. Virginia by clicking in the nav bar next to your name.
///image here

1. Once you're logged in and your data center is set to N. Virginia, head to the [AWS Console](https://console.aws.amazon.com) and type 'Lambda' into the search box at the top. Select the first option.

///image here


1. On the next page click on the 'Create a Lambda Function' button.

///image here


1. Here, you're going to want to select the "Blank Function option."
///image here


1. On the next page, the Configure triggers page, click on the grey box and select Alexa Skills Kit from the dropdown. Then click next. This tells Lambda that the function you're creating will be triggered by an Alexa request.
///image here

1. Next, under the configure function option, give your function a name, this can be anything you want. Optionally, give it a description. Make sure the runtime is set to "Node.js 4.3"

1. In the "Lambda function code" section, change the code entry type to be "Upload a .ZIP file" and then upload the .ZIP file we created earlier.

1. Further down the page, in the 'Lambda function handler and role' section, leave the Handler option set to 'index.handler.' In the role dropdown, select 'Create custom role' and click 'Allow' in the new tab that opens. Click 'Next' in the bottom right.

1. Now, on the review page you'll be able to see all the options you've selected. It should look something like this:
///image here

The click "Create function" in the bottom right corner.

1. Now, in the top right of the window, copy the ARN. You'll need to have this when you set up your skill in the Alexa developer portal.

## Step 3. Setting up your skill in the Alexa Developer portal
1. For this part you'll need to sit down with your mentor because you'll need access to the DevMountain amazon account in order to connect your Lambda function with the Alexas on campus.

1. Navigate to the [Amazon Developer Portal]('https://developer.amazon.com/') and login.

1. In the then head to the [Alexa section]('https://developer.amazon.com/edw/home.html#/') of the site by selecting "Alexa" in the nav bar.

1. On the 'Get Started with Alexa' page, select 'Get Started >' under the 'Alexa Skills Kit' option.

1. Next, on the top right, select 'Add a new Skill'
///image here

1. Fill out the next section to look just like the example below. You can fill in your own Name, and Invocation Name. The invocation name is what you'll say to alexa when you want your skill to be invoked, so make it short and easy to remember. Then select next.
///image here

1. On the next page, you're going to paste your Intent Schema into the section labeled "Intent Schema" that we created earlier.

1. Below that, you can ignore the "Custom Slot Types" section, and paste in your Sample Utterances into the appropriate section. Then click next.

///image here

1. Next, you'll be on the Configuration tab. Fill it out like below, and paste in the ARN from the top right of the page where you uploaded your zip file to AWS Lambda. It will start with 'arn:aws:lambda:us-east.' Then select next.

///image here

1. Now you're on the testing page. Here, in the section labeled 'Service simulator,' you can type in a phrase that your application is trained to accept. On the left side, you'll see the JSON that is sent to your program. On the right, you'll see your program's response. If you get an error, check the AWS console. Navigate to your Lambda function in AWS and click the "monitoring." On the right side, there should be a link to 'View Logs in CloudWatch.' From there you can see your error messages. Update your code accordingly, rezip the file, and upload it to AWS again.

///image here
///image here


## More resources
[Alexa NodeJS SDK]('https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs')
[Alexa Skills Kit Docs]('https://developer.amazon.com/alexa-skills-kit')
