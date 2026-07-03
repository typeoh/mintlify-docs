# Source: https://rasa.com/docs/learn/use-cases

On this page

### High-trust, fluent AI assistants[​](https://rasa.com/docs/learn/use-cases/#high-trust-fluent-ai-assistants 'Direct link to High-trust, fluent AI assistants')

Rasa assistants are great at handling tasks that require back-and-forth with a user. When real users interact with a chatbot or voice assistant, they rarely provide all the information you need in one go. Assistants have to ask clarifying questions, gather more information, query APIs, and follow dynamic branching logic to complete a task.

You can trust a Rasa assistant to handle that back and forth. Out of the box, Rasa already knows a lot about how conversations work: it automatically handles disambiguation, topic changes, clarification, corrections, implicatures, negation, interjections, and more. And while LLMs help power that out-of-the-box fluency, Rasa executes the steps in your task deterministically. So no matter how complex and high-stakes your business logic might be, Rasa will follow it faithfully.

Here are some of the conversation patterns that Rasa handles automatically:

- Clarification
- Implicit Answers
- Topic Changes
- Change of mind
- Interjections

User:

I lost my wallet

💡 Multiple flows could be relevant to this user, so Rasa asks for clarification

Bot:

Do you want to freeze your card or cancel it?

Bot:

Would you like to add anything else to your order?

User:

I've spent too much already!

💡 Set continue\_shopping to False and continue with the checkout process

Bot:

Can I interest you in next-day shipping for $4.99?

User:

I want to pay my credit card bill.

Bot:

Do you want to pay the whole balance of $3,421?

User:

how much do I have in my account?

💡 switching flows, show the user's balance

Bot:

your checking account balance is $1,304.21.

pick up where we left off

Bot:

how much would you like to pay into your credit card?

User: I need to send $50

Bot: Who would you like to send money to?

User: Jen

Bot:

Please confirm: you want to transfer $50.0 to Jen?

User: actually no 60

💡 the user changed their mind about how much money to send

Bot: Ok, I corrected the amount.

Bot:

Please confirm: you want to transfer $60.0 to Jen?

User: yes.

Bot: All done. $60.00 has been sent to Jen.

Bot: Is there anything else I can help you with?

Change of mind

Bot:

Would you like that gift wrapped?

User:

how much is that?

💡 the user is asking a question, consult the knowledge base

Bot:

As your order is over $25, our gift-wrapping service is free of charge!

pick up where we left off

Bot:

Shall I go ahead and get it wrapped?

### Real-time voice assistants that take action[​](https://rasa.com/docs/learn/use-cases/#real-time-voice-assistants-that-take-action 'Direct link to Real-time voice assistants that take action')

Rasa handles the core elements of spoken conversations. These include natural conversation elements like pauses and turn-taking, as well as phone-specific capabilities such as DTMF processing and call control. The platform integrates with speech recognition services and contact center systems for automating call center operations.

Rasa's dialogue understanding approach allows small LLMs (under 10B parameters) to match the accuracy of much larger models. These models can be self-hosted and respond quickly enough for real-time voice conversations. Here's a demonstration using a fine-tuned Llama 8B model:

### Multilingual, personalized support assistants that handle hundreds of tasks[​](https://rasa.com/docs/learn/use-cases/#multilingual-personalized-support-assistants-that-handle-hundreds-of-tasks 'Direct link to Multilingual, personalized support assistants that handle hundreds of tasks')

Each customer interaction is different, shaped by language, location, and context. With Rasa you can build personalized experiences using real-time contextual data about the user.

- Anisha
- Alex
- Ricardo

user profile loaded: language preference, subscription tier, recent activity

User:

you messed up my order

Bot:

Hi, Anisha! I'm sorry to hear that. I can see two recent orders - was the problem with the umbrella or with the kitchenware?

user profile loaded: language preference, subscription tier, recent activity

User:

you messed up my order

Bot:

Hi, Alex! I'm sorry to hear that. What was the issue with your microwave?

User:

I ordered it in black but the one you sent is grey

Bot:

Oh dear. Yes I can see that you ordered Combi Microwave 800W - Black.

Bot:

Sorry about the mix-up, we'll send out the right one by next-day delivery. You'll also get a return label to send back the grey one. Sorry for the inconvenience.

user profile loaded: language preference, subscription tier, recent activity

User:

😡

Bot:

Ricardo, je suis désolé d'apprendre que tu es malheureux. Comment puis-je t'aider ?

Rasa is especially helpful as you scale across domains and departments to cover the full breadth of support requests with:

- A centralized content hub for translation & localization
- Reusable dialog components across languages
- Collaborative workflows to review and update assistant versions.

| Order status | Refunds | Subscription Management | Product Search | Appointments | Complaints | ... |
| --- | --- | --- | --- | --- | --- | --- |

### Custom, controllable conversational AI[​](https://rasa.com/docs/learn/use-cases/#custom-controllable-conversational-ai 'Direct link to Custom, controllable conversational AI')

Rasa lets you build assistants where you need:

- Complete data control and privacy, with all processing happening in your infrastructure
- Full visibility into decision-making, with structured, traceable paths from input to output
- The ability to override & customize the AI engine of your assistant
- Deep integration with your existing systems
- A CI/CD-driven approach to deploying your assistant

With Rasa, you can:

- Self-host fine-tuned language models optimized for your domain
- Add custom components to modify any part of the conversation pipeline
- Build native integrations with your mobile apps and enterprise systems

### Ready to start?[​](https://rasa.com/docs/learn/use-cases/#ready-to-start 'Direct link to Ready to start?')

Check out the [Platform at a Glance](https://rasa.com/docs/learn/platform-introduction/)

- [High-trust, fluent AI assistants](https://rasa.com/docs/learn/use-cases/#high-trust-fluent-ai-assistants)
- [Real-time voice assistants that take action](https://rasa.com/docs/learn/use-cases/#real-time-voice-assistants-that-take-action)
- [Multilingual, personalized support assistants that handle hundreds of tasks](https://rasa.com/docs/learn/use-cases/#multilingual-personalized-support-assistants-that-handle-hundreds-of-tasks)
- [Custom, controllable conversational AI](https://rasa.com/docs/learn/use-cases/#custom-controllable-conversational-ai)
- [Ready to start?](https://rasa.com/docs/learn/use-cases/#ready-to-start)