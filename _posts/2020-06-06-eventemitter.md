---
title: Sharing State in React Using EventEmitter
categories: frontend
tags: react eventemitter javascript
---

![TitleImage](../assets/images/eventemitter.png)
{: style="width:10%,height:10%"}

## How do React Components share information among components? 
React libraries do not offer any global state store. 
*Then how do we share information between components?*
In a parent-child relationship, we pass the parent's state as props to child. For example, in an MCQ exam portal we may have components like Question(parent) and QuestionOptions(child). When a user selects an option, the state of Question(parent) changes. The selected Options component is notified through props that it needs to change its style to highlighted and then it is visible as a selected option to the user. 

Sharing state between child to parent components is not so easy in traditional React. We need to introduce a lot of callbacks which can make the code messy; especially if we need to share state between child and parent components farther up the chain.

## How do we make the sharing of state simpler?
We can see that we need to introduce a global state store in our application that acts as a single source of truth.
Simultaneously, it also needs to notify the associated components that state has changed and they need to re-render based on updated  values.

Redux is a popular framework used with React that helps in relaying global state information by firing "Action" events. But setting up a redux based store for smaller applications with few components is unnecessary.

## Introducing EventEmitter
EventEmitter is nothing but a simple publisher-subscriber model. If two components share state, then an event emitter can be used to establish a direct line of communication between them. It removes the overhead of passing state and callbacks to unrelated intermediate components.

It is designed along the principles of the Observer pattern. Within the context of React, this would mean some components subscribe to receive particular messages and other components publish messages to those subscribers. Components would typically subscribe in the **`componentDidMount`** method and unsubscribe in the **`componentWillUnmount`** method.

## How do we use it?
First, we need to add to eventemitter3 to our `package.json` file.
```javascript 
    npm install -save eventemitter3
```

For consistency in terms of publisher functions available throughout our code, we can make a separate class for EventEmitter.
A sample class can look like as below: 

```javascript
import EventEmitter from 'eventemitter3';

const eventEmitter = new EventEmitter();

const Emitter = {
    on: (event, fn) => eventEmitter.on(event, fn),
    once: (event, fn) => eventEmitter.once(event, fn),
    off: (event, fn) => eventEmitter.off(event, fn),
    emit: (event, payload) => eventEmitter.emit(event, payload)
}

Object.freeze(Emitter);

export default Emitter;
```

We can then import and use it in our classes as per our requirements. I'll demonstrate using a webRTC chat app that I built.

This is a function of Form.js component that emits(publishes) a *MESSAGE_RECEIVED* event whenever the user presses enter.
```javascript
 sendMessage = () => {
        let value = this.state.input;
        let payload = {
            user: "me",
            message: value
        }

        this.state.dataChannels.forEach(channel => {
            channel.send(value);
        });

        this.setState({ input: "" }
            , () => Emitter.emit(MESSAGE_RECEIVED, payload));
    }
```

The Message.js component has subscribed to the *MESSAGE_RECEIVED* event and changes its state whenever a new message comes and re-renders. The message is now visible to all the peers connected on the chat.

```javascript
import React from 'react';
import {MESSAGE_RECEIVED} from '../service/events.js';
import Emitter from '../service/emitter.js';
import {List, ListItem, ListItemText} from '@material-ui/core';

class Messages extends React.Component {


    constructor(props) {
        super(props);
        this.messagesEnd = React.createRef();
        this.state = {
            messages: []
        };
    }

    componentDidMount() {
        Emitter.on(MESSAGE_RECEIVED, payload => {
            this.setState({ messages: [...this.state.messages, payload] });
        });
    }

    render() {
        let newMessages = this.state.messages;
        return (
            <List>
                {
                    newMessages.map((message) =>
                        <ListItem>
                            <ListItemText
                                primary={message.message}
                                secondary={message.user}
                            />
                        </ListItem>
                    )
                }
                <div ref={this.messagesEnd}/>
            </List>
        );
    }
```

Thanks for reading. I hope this helps :) 

*Reference Links:* <br>
[Blog explaining Event Emitter in Detail](https://medium.com/@lolahef/react-event-emitter-9a3bb0c719) <br>
[Patterns to share state in React](https://www.javascriptstuff.com/component-communication/#6-observer-pattern) <br>
[GitHub Project Using EventEmiiter](https://github.com/priyapande/webRTC-signalling)