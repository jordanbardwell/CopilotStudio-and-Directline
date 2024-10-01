# CopilotStudio-and-Directline
I'm assuming if you've found this, the you're interested in how you can capture directline events from Copilot Studio. Low-code is cooler, but i'll still go over it.

## Handoff Activity
In Copilot Studio, you can Send an activity called `Handoff`. You could also use the `Transfer Conversation` option under topic management, which triggers a handoff.initate event. These events can be captured on the client and handled accordingly. You could either handle everything on the client, or you could re-direct it back to Copilot Studio topics.

### Capturing a Handoff Event with Javascript
Once an activity is sent such as Handoff, you need to capture it on the client (the users device). You can capture these events with Javascript that's located in the code that's used to render the bot. 

### What does the Javascript look like?
Here is a snippet of the Javascript that watches for the Handoff Event and dispatches another event back to the bot. When you trigger a Handoff event from a Copilot Studio topic, it can't see the handoff event itself, only the client does from my testing. 

So, you can dispatch another event back to the bot, for a `event received` topic to pickup. It also logs every incoming action:
```javascript
    const store = window.WebChat.createStore({}, ({ dispatch }) => next => action => {
        console.log('Action received:', action);

        // Listen for incoming activities to detect the handoff activity and custom event activities
        if (action.type === 'DIRECT_LINE/INCOMING_ACTIVITY') {
            const { activity } = action.payload;

            console.log('Incoming activity:', activity);

            // Detect the 'handoff' activity
            if (activity.type === 'handoff') {
                console.log('Handoff activity detected:', activity);
                dispatch({
                    type: 'DIRECT_LINE/POST_ACTIVITY',
                    meta: { method: 'keyboard' },
                    payload: {
                        activity: {
                            type: 'event',
                            name: "HandoffToTalkDesk",
                            from: { id: 'user' }
                        }
                    }
                });
                return;
            }
        }
        return next(action);
    });
```
The full code can be found in the HTML files in this repository.

# Let's break it down even further
I'm going to walk through how we could use client-side event capturing to detect a handoff event and dispatch an event back to Copilot Studio to trigger a topic.

## Understanding the `const store` in JavaScript

Imagine you have a mailbox called `store`. This mailbox is very smart and can do some cool things when it gets a letter (which we call an `action`).

### Creating the Mailbox
```javascript
const store = window.WebChat.createStore({}, ({ dispatch }) => next => action => {
```
Here, we’re making our mailbox (store). It’s set up to look at every letter (action) it gets.
### Reading the Letters
```javascript
console.log('Action received:', action);
```
### Checking the Letters
Every time a letter (action) comes in, the mailbox says, “Hey, I got a letter!” and shows us what’s inside.
```javascript
if (action.type === 'DIRECT_LINE/INCOMING_ACTIVITY') {
    const { activity } = action.payload;
    console.log('Incoming activity:', activity);
```
The mailbox looks at the letter to see if it’s a special kind called INCOMING_ACTIVITY. If it is, it opens the letter and checks what’s inside (activity).
### Finding Special Messages
```javascript
if (activity.type === 'handoff') {
    console.log('Handoff activity detected:', activity);
```
Inside the letter, if it finds a message that says "handoff", it says, "Handoff activity detected:" and shows the activity object.
### Sending a New Letter
```javascript
dispatch({
    type: 'DIRECT_LINE/POST_ACTIVITY',
    meta: { method: 'keyboard' },
    payload: {
        activity: {
            type: 'event',
            name: "HandoffToTalkdesk",
            from: { id: 'user' }
        }
    }
});
```
When it finds the "handoff" message, it writes a new letter saying, "Hey, let’s hand off to TalkDesk!" and sends it out.
### Passing Other Letters
```javascript
return next(action);
```
If the letter isn’t special, the mailbox just passes it along to the next mailbox.

So, your `store` is like a super-smart mailbox that reads every letter, looks for special messages, and sends out new letters when it finds something important. Pretty cool, right?
