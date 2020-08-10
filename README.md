# Reminder_Schedule_Cancel_bot


Oh.. I forgot to call you.

Oh.. I forgot that I had a meeting today.

Oh.. I forgot to attend the today’s session.

All of us may have said or have heard these kinds of sentences one or many times in our entire life. May be due to busy schedules or any other reason, and we must have lost many opportunities as well as relations. But not anymore, as I am back with a technical and the coolest solution for the same. Now, you can have a personal assistant in your pocket that can remind you all about meetings, calls, meets, attending the seminars, and many more. Sounds interesting.

So, today you will be learning all about how to build your personal assistant that can help you to remind you about many opportunities and meetups.

Let’s discuss what you will learn in today’s blog. You will learn,

– How to call the custom actions without any stories.

– How to setup the action to remind you to call anyone

Setting Up Custom Remind actions

Firstly, let’s start with setting up the actions for reminder. Before starting I assume that you already have gone through my blog to Build an end-to-end conversational bot with UI and you have a basic chatbot ready with you.

Now into that chatbot let’s add this simple conversation to see the working of the bot to remind you.

User: Hey buddy!
Bot: Hey, How can I assist you?
User: Remind me to call Ashish.
Bot: Sure, I’ll remind you in 15 seconds.

After, the time is up and the bot will have to remind you then the bot will reply automatically with,

Bot: This is a reminder to call Ashish.

Now, I think you have got the idea that what you have to do now. So firstly lets begin by adding the intent to the nlu.md file to accept the user input.

nlu.md

## intent:ask_remind_call
- remind me to call [Ashish]{"entity":"name","value":"Ashish"}
- remind me to call [Rahul]{"entity":"name","value":"Rahul"}
- later I have to call [Apneet]{"entity":"name","value":"Apneet"}
- later I have to call [Awakash]{"entity":"name","value":"Awakash"}

Now, add the intent and entity to the domain.yml file in the given way,

domain.yml

intents:
- ask_remind_call
entities:
- name

Now you have added the entity and intent into the domain file. But here we have to make changes to the domain file in writing the intent as now the bot will take an action automatically once it’s time to remind you. So, for that you have to write the given code for intents instead as per the actions.py script.

intents:
- ask_remind_call:
    triggers: action_set_reminder
- EXTERNAL_reminder:
    triggers: action_react_to_reminder
entities:
- name

Here, action_set_reminder and action_react_to_reminder are the custom actions that you will mention inside the actions.py script as shown here.

actions.py

import datetime
from rasa.core.events import ReminderScheduled
class ActionSetReminder(Action):
"""Schedules a reminder, supplied with the last message's entities."""

 def name(self) -> Text:
return "action_set_reminder"

 async def run(
self,
dispatcher: CollectingDispatcher,
tracker: Tracker,
domain: Dict[Text, Any],
) -> List[Dict[Text, Any]]:

dispatcher.utter_message("I will remind you in 15 seconds.")

date = datetime.datetime.now() + datetime.timedelta(seconds=15)
entities = tracker.latest_message.get("entities")

reminder = ReminderScheduled(
"EXTERNAL_reminder",
trigger_date_time=date,
entities=entities,
name="my_reminder",
kill_on_user_message=False,
)

return [reminder]

class ActionReactToReminder(Action):
"""Reminds the user to call someone."""

 def name(self) -> Text:
return "action_react_to_reminder"

 async def run(
self,
dispatcher: CollectingDispatcher,
tracker: Tracker,
domain: Dict[Text, Any],
) -> List[Dict[Text, Any]]:

name = next(tracker.get_latest_entity_values("name"), "someone")
dispatcher.utter_message(f"This is a Remember to call {name}!")

return []

Here, you have two actions one is action_set_reminder which will take the input from the user and extract the entities and store it like in our case the entity names. So, that name will be picked and will be used to schedule the reminder according to the user name. For now, we are using the default time to set the reminder so the reminder is set after 15 seconds with respect to the datetime module. A reminder schedule object is created with the given code to send multiple arguments like intent name, trigger date-time, entities, etc which will schedule the reminder for the time after 15 seconds.

reminder = ReminderScheduled(
"EXTERNAL_reminder",
trigger_date_time=date,
entities=entities,
name="my_reminder",
kill_on_user_message=False,
)

On the basis of this intent name “EXTERNAL_reminder” which we specify here next custom action “action_react_to_reminder” will be triggered automatically when it’s time to remind you to make a call. Here kill_on_user_message is used a a flag which will be used to enable/disable the conversation after you have scheduled the reminder and you are busy with some other works.

Note: The reminder that you have scheduled will be active until your rasa server is active.

Also, add these action names to the domain file to make it work.

domain.yml

actions:
- action_set_reminder
- action_react_to_reminder

Now, everything is set up and you can now test your bot on Rasa X (recommended). You may have noticed that we haven’t created any story for it as we are using the trigger method in the domain file which will automatically link the intents with the respective actions.

You can also check this session for the more clarification,
Subscribe to Innovate Yourself

Now run the given commands in seperate terminals and test you bot,

rasa x
rasa run actions

This is how your output will looks like,

This is all about this blog, I hope you have enjoyed this blog. But still, if you are having any queries regarding the topic or you want to share your views then feel free to leave a comment below in the comment section.
Stay tuned and Happy Learning. 🙂
