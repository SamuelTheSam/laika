# Laika

Laika is an open-source personal assistant made for developers, designed to be easy to deploy and adapt.

**[Leia-me em português](https://github.com/felipelima555/laika/blob/master/README_pt-BR.md)**

Basically, using dependency injection and some decorators, it is extremely simple to add new features to Laika. The ```@MessageHandler``` decorator injects the metadata that teach Laika how to respond to your messages.

<img src="https://user-images.githubusercontent.com/20775579/90994643-c10a8e80-e58f-11ea-9949-ac5594e09fc4.png" />

<img src="https://user-images.githubusercontent.com/20775579/90994647-c7990600-e58f-11ea-898f-c90aa748f221.gif" />

## Getting started 

```bash
git clone https://github.com/felipelima555/laika
cd laika
yarn install
docker-compose up
```

Docker Compose will automatically set up the entire environment that Laika needs to work, containing the backend (NestJS), the frontend (React) and the database server (MongoDB). For more information see [docker-compose.yml](https://github.com/felipelima555/laika/blob/master/docker-compose.yml).

**After starting the environment, open your browser, go to http://localhost:3000 and say "Hello" to Laika!**

*NOTE: If you prefer not to use Docker, you can directly start the backend with the command `yarn start:dev` (in the server directory). For the frontend, open another terminal and run `yarn start` (in the client-web directory). In this case, you will need to have a MongoDB server active at localhost:27017.*

Here are some things you can say to Laika:

- Who are you?
- How are you?
- Tell me a joke
- I'm bored, tell me what to do
- Save the note buy milk
- Show all my notes
- Show notes about milk

## Technologies

### Frontent (React)
- **styled-components** for styling
- **socket-io.client** to send / receive messages from the server
- **React Context API** for global state management.

### Backend (NestJS)
- **socket-io** for communication with the frontend
- **node-nlp** for natural language processing
- **mongoose** for the MongoDB database
- **reflect-metadata** to add and get metadata using custom decorators
- **rxjs** to stream messages with Observables

The backend can be understood through the two main modules:

- **src/core** - Here are the message processing and training of the artificial intelligence algorithm. There are also decorators that will be used in the skills directory.
- **src/skills** - Here are the modules that tell Laika what to do. Using the decorator ```@MessageHandler``` which is exported from the src/core folder, the function will be collected by the core and used in the processing of messages.

## Add new skills
The coolest part about Laika is that it’s extremely easy to add new skills. Just use the NestJS modules/services system. Start with the commands below:

```bash
cd server
yarn nest generate module skills/my-awesome-skill
yarn nest generate service skills/my-awesome-skill
```

NestJS will create the directory ```skills/my-awesome-skill```, containing a *module* and a *service*, and will inject the new module into the facilities of ```skills.module.ts```. All we need to do now is edit our `my-awesome-skill.service.ts` file.

Now let's import the @MessageHandler decorator from ```src/core``` to teach Laika a new function. Copy and paste the example below into your `my-awesome-skill.service.ts` file:

```typescript
import { Injectable } from '@nestjs/common';

import { MessageHandler } from 'src/core';

@Injectable()
export class MyAwesomeSkillService {

  /**
   * Example 1
   * The @MessageHandler decorator is used to inform the core
   * to add the phrase and that function to message processing.
   * In that case, when you say "this is a test",
   * Laika will answer "It works!"
   */
  @MessageHandler('This is a test')
  testing() {
    return `It works!`;
  }

}
```

Save the file and the server will automatically restart, learning the new features. Then, return to the browser and test the new function.

![test](https://user-images.githubusercontent.com/20775579/90997595-5ad63980-e598-11ea-9b57-41f9f069b70d.gif)

You can add multiple functions in the same class, just use the @MessageHandler decorator and Laika will learn them. Try copying and pasting the code below into your ```my-awesome-skill.service``` for more examples.

```typescript
import { Injectable } from '@nestjs/common';
import { range, of } from 'rxjs';
import { delay, concatMap } from 'rxjs/operators';

import { MessageHandler, MessageText, TrimEntity } from 'src/core';

@Injectable()
export class MyAwesomeSkillService {

  /**
   * Example 1
   * The @MessageHandler decorator is used to inform the core
   * to add the phrase and that function to message processing.
   * In that case, when you say "this is a test",
   * Laika will answer "It works!"
   */
  @MessageHandler('This is a test')
  testing() {
    return `It works!`;
  }

  /**
   * Example 2
   * You can train multiple sentences for the same function.
   */
  @MessageHandler(['Say my message', 'Repeat my message'])
  repeatMessage(
    // Use the @MessageText decorator to inject your own message
    @MessageText() myMessage: string,
  ) {
    // Returning an array, a random response will be chosen.
    return [
      `You said: ${myMessage}`,
      `Your message was "${myMessage}"`,
    ];
  }

  /**
   * Example 3
   * You can return an object containing the text and a function
   * to create more complex dialogues
   */
  @MessageHandler('Say my name')
  sayMyName() {
    return {
      text: `What's your name?`,
      nextCallback: (text: string) => `Your name is ${text}`,
    };
  }

  /**
   * Example 4
   * This is a slightly more complex example
   * In this case, if you say "Count from 1 to 10" Laika will
   * reply with a stream of 10 messages.
   */
  @MessageHandler('Count from %n1% to %n2%')
  count(
    // You can extract data with the @TrimEntity decorator
    @TrimEntity('n1', { between: ['from', 'to'] }) n1?: string,
    @TrimEntity('n2', { after: 'to' }) n2?: string,
  ) {
    const fromNumber = parseInt(n1, 10) || 0;
    const toNumber = parseInt(n2, 10) || 0;
    const response$ = range(fromNumber, 1 + toNumber - fromNumber)
      .pipe( concatMap(n => of(n).pipe(delay(500))) );

    // Return an Observable <string> to create a message stream
    return response$;
  }

}
```

Save the file and the server will automatically restart, learning the new features. Then, return to the browser and test some of the phrases.

**Now you're ready to start creating your own personal assistant!**

Take a look at the other directories in src / skills for more examples.

## Contributing
Laika is still in development and any contribution is welcome! Below is a list of what I would like to improve:

## TODO
- Show old messages when opening the app
- Speech to text and text to speech
- Docker Swarm for deploy
- Exception handling
- Automated testing
- Hot word (activate with the voice command "Hey Laika!")
- Transform the React app into a progressive web app (PWA)
- Notifications of new messages, even when the app is out of focus

## Author
I'm **Felipe Lima** and you can contact me through my [LinkedIn](https://www.linkedin.com/in/felipelimadasilva/).