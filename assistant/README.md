# Creating a Google Assistant To-do application with Google Home, Spring Boot and Polymer

**This workshop uses three prepared repos:**

- https://github.com/effibennekers/todo-api
- https://github.com/effibennekers/todo-fe
- https://github.com/Chapter-meeting/chapter-meeting.github.io

We will be making use of a prepared google account:
chapter.dev.eng.2@gmail.com / redherring

Static content is served on [GitHub pages](https://chapter-meeting.github.io/)


## Google Assistant on your Google Home
### Create an Actions project and Dialogflow agent

To develop for Actions on Google, you need to create an Actions project and Dialogflow agent. The Actions project allows lets you register your actions, manage your app through the approval process, view analytics for your app, and more. The Dialogflow agent defines what users can say to your app and how your app processes users input.

1. Go to the [Actions on Google Developer Console](https://console.actions.google.com/u/0/).
- Click on Add Project, enter TodoMaker for the project name, and click Create Project.
- In the Overview screen, click BUILD on the Dialogflow card and then CREATE ACTIONS ON Dialogflow to start building actions.
- The Dialogflow console appears with information automatically populated in an agent. Click Save to save the agent.
- Your agent calls some fulfillment logic that you'll build later to return a response with the user's silly name. Enabling fulfillment for your agent enables some UI fields that we'll need later, so let's do this now:
 - Click the Fulfillment menu item.
 - Move the Disabled slider for Webhook to the right to Enabled.
 - Enter a dummy URL for now, https://todo.maker. We'll update this URL when you build and deploy your fulfillment.
- Click Save.

### Define actions and invocation

When you create a Dialogflow agent, a Default Welcome intent is automatically created. This intent represents the main entry point into your conversation or main action of your app. All your apps must have this main action defined, so that Actions on Google knows how to invoke your app. This intent has the following characteristics:

- The Events section of the intent specifies a WELCOME event, which signifies that this intent is the default entry point into your app. Each agent must have one and only one intent that declares this WELCOME event. The Google Assistant uses this event to trigger your app when users invoke your app by name, such as with "Ok Google, talk to Todo Maker".
- The Action field specifies an input.welcome name. When an intent is triggered, this action name is sent to your fulfillment so it can map the action's name to the task to carry out. Because this welcome intent doesn't need to use fulfillment (all it does is respond with a static response), you can leave this as-is, because it's never used.
*Note: A Dialogflow action has a different meaning than an Actions on Google action, so don't confuse the two as being the same thing. A Dialogflow action is a component of an intent that defines the data to act upon. Actions on Google actions are entry points into your apps.*
- In the Response section, there are default, static text responses. This lets you respond to users without a call to your fulfillment. The Default Welcome Intent contains pre-populated responses that you should remove to add your own.

*Dialogflow responses versus fulfillment responses
Dialogflow lets you create an app without having any fulfillment at all, because it lets you define responses directly within Dialogflow without writing fulfillment code. This is convenient for simple messages that don't require any logic to generate. However, most of your intents will probably use fulfillment to return a response.
Another useful way to use the Response feature in Dialogflow is to return "placeholder" responses during development. This lets you ensure that an intent is properly triggered with the grammar that you define.*

Now that you know a little about the welcome intent, lets modify it to suit our needs:

1. In the left-navigation menu, click Intents and select the Default Welcome Intent.
2. In the Response section, delete the pre-populated responses in the Text response fields by clicking on the trash can icon in the upper-right corner.
3. Click on ADD MESSAGE CONTENT > Text response, specify the following response instead, and click Save: **Hi! Welcome to your Todo list! Let's get started. How many todos do you want to hear?**
4. Click save.

Now, when users invoke your app, they know that they're entering your app's experience and what to say next. In general, your responses should guide users on what to say next and to stay within your conversations' grammar, which we'll further define in the next section.

### Define your conversation's grammar

Dialogflow intents define your conversation's grammar and what tasks to carry out when users speak particular phrases of your grammar. Because Dialogflow includes a natural language understanding (NLU) engine, you don't need to be exhaustive when defining the phrases; Dialogflow automatically expands on the phrases you provide and understands many more variations of them.

Create an intent that defines how users need to provide the amount of todos they want to hear.

1. In the left navigation, click the + icon by the Intents menu item.
2. In the Intent name field, enter todo_amount.
3. In the User says field, enter `I want 3 todos` and hit Enter.
4. Click and highlight the number `3` and assign it the @sys.number entity in the drop-down menu that appears. Dialogflow might try and intellgiently assign this automatically to another entity, such as @sys.age. You can always change it's guess to another entity.
5. Provide more variations for the User says phrases and assign any number within those phrases the @sys.number entity:
 - 3
 - 3 todos
6. In the Actions section, select the REQUIRED checkbox for the number parameter. This tells Dialogflow to not trigger the intent until the parameter is properly provided by the user.
7. Click on Define prompts for the parameter and provide a reprompt phrase. This phrase is spoken to the user repeatedly until the Dialogflow detects a number in the user input. Enter "How many todos do you want to get?"
8. In the Fulfillment section, select the Use webhook checkbox. This tells Dialogflow to call your fulfillment to generate a response to the user instead of using Dialogflow's response feature.
9. In the Google Assistant section, select the End conversation checkbox. This tells Dialogflow to relinquish control back to the Google Assistant after your fulfillment returns a response to the user.
10. Click SAVE to save the entire intent.

That's it! You've successfully declared your conversation's grammar with Dialogflow intents. You've also used Dialogflow's built-response feature to return a static response to the user when they invoke your app. You also created an intent that uses fulfillment to return a response, which you'll now need to build. This takes the form of an API and a web frontent built in Polymer.

### Preview the app
Now that you have your fulfillment deployed and properly set in Dialogflow, you can preview the app in the Actions simulator.

To preview your app:
1. Optional: Turn on the Web & App Activity, Device Information, and Voice & Audio Activity permissions on the [Activity controls](https://myaccount.google.com/activitycontrols) page for your Google account. You need to do this to use the Actions Simulator, which lets you test your actions on the web without a hardware device.
2. Click on Integrations in Dialogflow's left navigation, and move the slider bar for the Google Assistant card if not already active.
3. Click on the Google Assistant card and click on TEST in the pop-up window. Dialogflow uploads the agent to our servers so you can test it out in the Actions simulator.
4. In the Test your Assistant app section of the integration screen, click on the Actions Simulator link to open a new browser for the Actions simulator.
5. Make sure your API is reachable.

You will need the url of the API that is built and put it in the fulfillment webhook.

## API

To have Actions on Google talk to your API, you need to make it available to the outside world. If you want to expose a local server behind a NAT or firewall to the internet, we need a secure tunnel to localhost (the Pi cluster). Ngrok has already been installed.

Check out the project at [GitHub](https://github.com/effibennekers/todo-api) by cloning git@github.com:effibennekers/todo-api.git or https://github.com/effibennekers/todo-api.git. Make sure you have access.

Now we create an API using spring initalizr plugin from IntelliJ.
Add an POST endpoint at /todos, taking a String
Print it out in the console to see whether you get anything in.

Example:

	@RestController
	public class HelloController {
      @RequestMapping(value = "/todos", method = RequestMethod.POST)
      public String index(@RequestBody String body) {
        System.out.println(String.format("getting %s", body));
        Example request = new ObjectMapper().readValue(body, Example.class);
        final String resolvedQuery = request.getResult().getResolvedQuery();
        return new WebhookResponse(String.format("Oh my gracious! %s is a lot!", resolvedQuery) , "test");
      }
    }
  
Of course, this is not enough. You will need to marshall correct values.
The JSON string looks like this:

    {"originalRequest":{"source":"google","version":"2","data":{"isInSandbox":true,"surface":{"capabilities":[{"name":"actions.capability.AUDIO_OUTPUT"},{"name":"actions.capability.SCREEN_OUTPUT"}]},"inputs":[{"rawInputs":[{"query":"3","inputType":"KEYBOARD"}],"arguments":[{"rawText":"3","textValue":"3","name":"text"}],"intent":"actions.intent.TEXT"}],"user":{"lastSeen":"2017-11-14T10:38:17Z","locale":"en-US","userId":"ABwppHFnyyW9N3Np23CiGnhlMuOhK1dxdNGSUdkzSFK-_ECphhhWV0pCQAJb2SxJ3eI0xff9tZ_pYkpaOkI"},"conversation":{"conversationId":"1510656204630","type":"ACTIVE","conversationToken":"[]"},"availableSurfaces":[{"capabilities":[{"name":"actions.capability.AUDIO_OUTPUT"},{"name":"actions.capability.SCREEN_OUTPUT"}]}]}},"id":"b4cefc80-7daf-4774-b76e-9106bbe6a366","timestamp":"2017-11-14T10:44:12.29Z","lang":"en-us","result":{"source":"agent","resolvedQuery":"3","speech":"","action":"","actionIncomplete":false,"parameters":{"number":"3"},"contexts":[{"name":"actions_capability_screen_output","parameters":{"number":"3","number.original":"3"},"lifespan":0},{"name":"actions_capability_audio_output","parameters":{"number":"3","number.original":"3"},"lifespan":0},{"name":"google_assistant_input_type_keyboard","parameters":{"number":"3","number.original":"3"},"lifespan":0}],"metadata":{"matchedParameters":[{"required":true,"dataType":"@sys.number","name":"number","value":"$number","prompts":[{"lang":"en","value":"How many todos?"}],"isList":false}],"intentName":"todo_amount","intentId":"b5f17ff6-5892-495d-9042-edce5dfa8813","webhookUsed":"true","webhookForSlotFillingUsed":"false","nluResponseTime":23},"fulfillment":{"speech":"","messages":[{"type":0,"speech":""}]},"score":1.0},"status":{"code":200,"errorType":"success"},"sessionId":"1510656204630"}

  
Tip: use [json2schema](http://www.jsonschema2pojo.org/) to gemerate your domain objects from this JSON.

Parse the given number, and return it in a correct WebhookResponse format. Example:


    public class WebhookResponse {
      private final String speech;
      private final String displayText;

      private final String source = "todos-webhook";

      public WebhookResponse(String speech, String displayText) {
        this.speech = speech;
        this.displayText = displayText;
      }

      public String getSpeech() {
        return speech;
      }

      public String getDisplayText() {
        return displayText;
      }

      public String getSource() {
        return source;
      }
    }

In order to call this api, use ngrok. Run it on the same Pi as your API.

This is the first roundtrip. Think about adding, finishing, decorating todos. Align with the Google Assistant implementation and Frontend implementation.

## Frontend

We are going to use GitHub pages for showing our frontend. To do this, an organization called chapter-meeting has been created.
In the todo-fe project, create your Polymer app. After building, you can put the static content in the chapter-meeting.github.io repository. Check the same url http://chapter-meeting.github.io and see the results!

There are two ways in which you can get content from your API. The first one is just calling the Ngrok address exposed by the API. The second one takes more effort and is more complicated: use FireBase as an intermediary for your data. Polymer has excellent support for getting information updates from FireBase.

Create a list of todos, showing the api results. Decide on what a todo looks like together with the API creators.
