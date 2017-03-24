# Building an Alexa Skill

With the Alexa Skills kit you can make a program that responds to requests from an Alexa device. Here, you'll learn how to build an app using Node that can handle these requests. Then, you'll learn how to create an AWS Lambda function to host your skill and get it connected to your Alexa.

## Step 1\. Writing the code for your Alexa Skill

We're going to be using Amazon's <a href='https://www.npmjs.com/package/alexa-sdk' target="blank">alexa-sdk</a> for Node to write our application.

1. Set up a new project. Make a new folder called HelloAlexa.

2. Go into your new directory and run `npm init -y` in the terminal. Then run `npm install --save alexa-sdk` to save the Alexa SDK as a dependency.

3. Now that you've set up your dependencies, create an `index.js` file and open it up.

4. Start out in `index.js` by importing the Alexa package we just downloaded.

	`var Alexa = require('alexa-sdk')`
5. First, we're going to create the Intent Schema for our action. You shouldn't include this in your ```index.js``` file since we'll be copying it into a form on the Alexa developer site later. For now, just save it in a file somewhere else on your computer. The Intent Schema is a JSON object that tells the Alexa service the actions our application will handle. The schema below defines a skill that has two intents, a HelloWorldIntent, and a GetWeatherIntent. The GetWeatherIntent has an additional section called slots, which tells us that this intent will require information from the user (in this case a city) in order to run. In order to define a slot, you must give it a name and a type. Amazon has <a href='https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/built-in-intent-ref/slot-type-reference' target='blank'>a ton of premade types</a>, so its best to select one from that list when defining your slots. Here is the Intent Schema for the skill we're going to make in this project:

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

6. You'll also need to define a list of Sample Utterances that will be used to invoke your program. The more examples you provide, the more accurate Alexa will be at launching your skill at the correct times. For the intent that has a slot, we need to provide the name of the slot in the format shown above. When defining the sample utterances, you don't need to worry about punctuation. Save this along with your intent schema as a separate file somewhere. You'll need it later when you sit down with your mentor to set up the alexa skill on the developer portal.Here is a list that would be applicable to the intent schema shown above.

    ```
    HelloWorldIntent say hello
    HelloWorldIntent tell me hello
    GetWeatherIntent get me the weather in {city}
    GetWeatherIntent how is the weather in {city}
    GetWeatherIntent whats the weather like in {city}
    GetWeatherIntent what is the temperature in {city} today
    GetWeatherIntent how is the weather today in {city}
    ```

7. Back in ```index.js```, the first part of the application we'll want to implement are the event handlers. We defined the events our app will handle in the Intent Schema we defined above. The handler object we create here will contain a key for each event we defined earlier. Here is an example that would work with the demo JSON file from the previous section:

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

             //this http request wouldn't actually work because we didn't install a dependency
             //to make requests. Install whatever request library you prefer and make your request here.
             request('weatherAPIUrl').then(function(response){
                    temp = response.data.temperature;
                    this.emit(':tell', 'The temperature is ' + temp);
             });
         }
     }
    ```

    In order to have Alexa report back to the user, we use the `this.emit` function. The first parameter passed in (in this case, we used `':tell'`), tells the service how to respond to the user. If you use `':tell'`, Alexa will speak the text you pass in. If you pass in `':tellWithCard'` a card will show up in the Alexa app with the information you pass in. For now, let's stick to using `':tell'`.

8. Next, we need to register the `handlers` object we created above with the Alexa SDK. The code below tells the application what handlers have been defined and allows requests our app receives to be routed to the correct handler. In the last line, we tell the app to actually run with `alexa.execute()`.
  ```
  exports.handler = function(event, context, callback){
      var alexa = Alexa.handler(event, context);
      alexa.registerHandlers(handlers);
      alexa.execute();
  }
  ```
9. Now the app is ready to be uploaded to AWS. In the next step, we'll go over creating a function in AWS Lambda. But first, find your project folder on your computer and open it. Then compress everything inside the folder to a .zip file. It's extremely important that you open the file and the compress the contents, rather than just compressing the HelloAlexa folder, as that is the only way your function will work when you upload it to AWS.   
<p align="center">
    <img src="http://imgur.com/rCikqF2.png" width="450"/>
</p>

## Step 2. Setting up AWS Lambda

1. Go to <a href='https://aws.amazon.com' target='blank'>AWS</a> and sign up for a free account if you don't already have one. The free tier includes 1 million requests per month for one year, so that should be plenty.

2. When making your account select 'N. Virigina' as your data center. This is the only data center that currently supports Alexa requests. If you already have an account, log in and change your data center to N. Virginia by clicking in the nav bar next to your name.   
<p align="center"><img src="http://imgur.com/pgj9fQ1.png" width="300"/></p>
3. Once you're logged in and your data center is set to N. Virginia, head to the <a href='https://console.aws.amazon.com' target='blank'>AWS Console</a> and type 'Lambda' into the search box at the top. Select the first option.   
<p align="center">
    <img src="http://imgur.com/kZHMhFz.png" width="500" />
</p>

4. On the next page click on the 'Create a Lambda Function' button.     
<p align="center">
    <img src="http://imgur.com/IM4XWkJ.png" width="200"/>
</p>

5. Next, you're going to want to select the "Blank Function option."   
<p align="center">
    <img src="http://imgur.com/aITewUu.png" width="300" />
</p>

6. On the next page, the Configure triggers page, click on the grey box and select Alexa Skills Kit from the dropdown. Then click next. This tells Lambda that the function you're creating will be triggered by an Alexa request.
![](http://imgur.com/EIVclN4.png)

7. Next, under the configure function option, give your function a name, this can be anything you want. Optionally, give it a description. Make sure the runtime is set to "Node.js 4.3" (or whatever the latest version of Node is in the list of options).

8. In the "Lambda function code" section, change the code entry type to be "Upload a .ZIP file" and then upload the .ZIP file we created earlier.

9. Further down the page, in the 'Lambda function handler and role' section, leave the Handler option set to 'index.handler.' In the role dropdown, select 'Create custom role' and click 'Allow' in the new tab that opens. Then click 'Next' in the bottom right.

10. Now, on the review page you'll be able to see all the options you've selected. It should look something like the image below. The click "Create function" in the bottom right corner.   
<p align="center">
    <img src="http://imgur.com/b7whaJf.png" width="600" />
</p>

11. Now, in the top right of the window, copy the ARN and save it somewhere. You'll need to have this when you set up your skill in the Alexa developer portal.

## Step 3. Setting up your skill in the Alexa Developer portal
1. For this part you'll need to sit down with your mentor because you need access to the DevMountain Amazon developer account in order to connect your Lambda function with the Alexas on campus.

2. Navigate to the <a href='https://developer.amazon.com/' target='blank'>Amazon Developer Portal</a> and login with the DevMountain Amazon developer account.

3. Then then head to the <a href='https://developer.amazon.com/edw/home.html#/' target='blank'>Alexa section</a> of the site by selecting "Alexa" in the nav bar.

4. On the 'Get Started with Alexa' page, select 'Get Started >' under the 'Alexa Skills Kit' option.
![](http://imgur.com/kVlnUjG.png)

5. Next, on the top right, select 'Add a new Skill'   
<p align="center">
    <img src="http://imgur.com/fl33evf.png" width="200" />
</p>

6. Fill out the next section to look just like the example below. You can fill in your own Name, and Invocation Name. The invocation name is what you'll say to alexa when you want your skill to be invoked, so make it short and easy to remember and understand. Then select next.
![](http://imgur.com/3tClT4n.png)

7. On the next page, you're going to paste your Intent Schema that we created earlier into the section labeled "Intent Schema". You can ignore the "Custom Slot Types" section. Then paste in your Sample Utterances into the appropriate section below. Then click next.   
<p align="center">
    <img src="http://imgur.com/qviAzMB.png" width="500" />
</p>

8. Next, fill out the configuration tab like below, and paste in the ARN for your Lambda function. The ARN tells Alexa what server to route requests to your app to. You can find it by logging in to your AWS account, going to your Lambda function and looking in the top right corner of the page. It should start with 'arn:aws:lambda:us-east.' Once you've completed this form select next.
![](http://imgur.com/ipZuBmX.png)

9. Your Alexa app is now ready to be tested! You should be able to use your app on DevMountain's Alexas now. You can also test your app here on the testing page. Here, in the section labeled 'Service simulator,' you can type in a phrase that your application is trained to accept. On the left side, you'll see the JSON that is sent to your program. On the right, you'll see your program's response. If you get an error, check the AWS console. To see the console, navigate to your Lambda function in AWS and click on the "monitoring" tab. On the right side, there should be a link to 'View Logs in CloudWatch' (see the second image below). From there you can see your error messages. Update your code accordingly, rezip the file, and upload it to AWS again.     
<p align="center">
    <img src="http://imgur.com/ej8xtfJ.png" width="600" />
</p>
<p align="center">
    <img src="http://i.imgur.com/MeujO69.png" width="600" />
</p>

## More resources
[Random Fact Alexa App Tutorial by Amazon](https://github.com/alexa/skill-sample-nodejs-fact)   
[Alexa NodeJS SDK on Github](https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs)   
[Alexa Skills Kit Docs](https://developer.amazon.com/alexa-skills-kit)
