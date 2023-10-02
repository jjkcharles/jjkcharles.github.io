---
layout: post
title: TypeChat
subtitle: Harnessing LLMs for Type-Safe Natural Language processing!
author: jjk_charles
categories: explorations
tags: chatgpt llm explorations  
---

Ever since Microsoft announced [TypeChat library](https://microsoft.github.io/TypeChat/), I've been eager to try it. Not solely due to the current hype around Large Language Models (LLM), but because it addresses some intriguing challenges that were previously daunting or unwieldy.

## What is TypeChat?
As per the [blog post](https://microsoft.github.io/TypeChat/blog/introducing-typechat/) announcing TypeChat:
> ... how do we augment traditional UI with natural language interfaces? How do we use AI to take a user request and turn it into something our apps can operate on? And how do we make sure our apps are safe, and doing work that developers and users alike can trust?
> 
> Today we're releasing TypeChat, an experimental library that aims to answer these questions. It uses the type definitions in your codebase to retrieve structured AI responses that are type-safe.

As its called out, the key here is the ability for Natural Language inputs to be converted into Type safe objects which then could be used for integrating systems (either via API calls, messages or something else).

## What is the usecase?
The [examples](https://github.com/microsoft/TypeChat/tree/main/examples) accompanying TypeChat repo has good coverage on some of the use cases where something like TypeChat makes sense.

However, in general, any scenario in which you need to process natural language input from users and take corresponding actions within your application is a suitable use case.

Consider creating something akin to the following examples:
1. Chatbot - Building a chatbot requires building whole lot of decision trees to cover all the scenarios, and even then it might not be able to meaningfully parse all the input users type in. Imaging user's input such as "set reminder for every thurs at 7 PM", "set reminder for every Thursday @ 7:00 pm", "add an event for 7 pm 4 days from now" 
2. Natural language Search capability on an eCommerce applications - There are capabilities within search engines to handle synonyms, minor corrections on typos, phonetic matches etc. but they still require lots of upfront data setup and also the end result cannot be good enough unless we put in a customized solution that can handle the various kinds of inputs system could be receiving.

## What problem does it solve?
What if there is an easier way for us to be able to parse input received in Natural Language, and convert it into well-known concrete objects that systems can easily process.

Consider this example input:
> laptops with 13 inch displays an i7 processors

Directly feeding such input into a system and achieving meaningful processing is far from straightforward.

On the other hand, consider if we could expose an API conforming to a well-defined schema, like the one below, and the input can somehow be translated to such a structured manner, it removes many of the hurdles in being able to process data received in natural language.

This is where TypeChat comes into play, where you pass it with a "type definition" and also the input string, and it brokers with LLMs (OpenAPI ChatGPT and Azure OpenAI are supported out of the box) to best convert the input into something that fits into target "type".

```json
{
    ...
    "productType": "laptop",
    "techSpecs": {
        ...
        "displaySize": 13,
        "cpu": "i7"
        ...
    }
    ...
}
```

## Running the examples
TypeChat Github repo comes with a few examples already, and that might be the best way to test waters with this library. To get the examples running follow the [steps provided here](https://github.com/microsoft/TypeChat/tree/main/examples#step-1-configure-your-development-environment).

1. Install [Node.js & npm](https://nodejs.org/en/download)
2. Cloning the TypeChat repository - https://github.com/microsoft/TypeChat
3. Creating .env file at the root of the repo with Model specific details. I was pointing to OpenAI Model, so was using a config like below,
```
OPENAI_MODEL=gpt-3.5-turbo
OPENAI_API_KEY=sk-kjlsdfn94w5rkelnfsdfskhknwer4350dnksldff
OPENAI_ENDPOINT=https://mockgpt.wiremockapi.cloud/v1/chat/completions
```
4. Build the project as below,
```bash
npm run build-all
```
5. Navigate into one of the examples folders and run them as below. I am executing the Sentiment example with interactive input
```bash
cd .\examples\sentiment
node .\dist\main.js
```
You'll encounter a prompt to input text, and pressing Enter will lead TypeChat to interact with LLM, ultimately providing you with an output:
```
😀> I am feeling good today!
The sentiment is positive
```

> You might have noticed that I'm not utilizing [OpenAI's API endpoint](https://platform.openai.com/docs/api-reference) here; I'll elaborate on this later. 

## Analyzing an example
When examining the examples, you'll find just two major files to concentrate on (examples like "music" could be a bit more advanced, but the core concept remains the same).

I'll again be using the Sentiment example here.

The Schema file (sentimentSchema.ts):
```typescript
export interface SentimentResponse {
    sentiment: "negative" | "neutral" | "positive";  // The sentiment of the text
}
```
Here, we indicate the structure of the expected and also indicate the possible values for "sentiment".

```typescript
import fs from "fs";
import path from "path";
import dotenv from "dotenv";
import { createLanguageModel, createJsonTranslator, processRequests } from "typechat";
import { SentimentResponse } from "./sentimentSchema";

// TODO: use local .env file.
dotenv.config({ path: path.join(__dirname, "../../../.env") });

const model = createLanguageModel(process.env);
const schema = fs.readFileSync(path.join(__dirname, "sentimentSchema.ts"), "utf8");
const translator = createJsonTranslator<SentimentResponse>(model, schema, "SentimentResponse");

// Process requests interactively or from the input file specified on the command line
processRequests("😀> ", process.argv[2], async (request) => {
    const response = await translator.translate(request);
    if (!response.success) {
        console.log(response.message);
        return;
    }
    console.log(`The sentiment is ${response.data.sentiment}`);
});
```

Here the key piece of code you would have to be looking into is, creating the model and creating a translator, which will then be used to translate the "input" data.

```typescript
...
const model = createLanguageModel(process.env);
...
const translator = createJsonTranslator<SentimentResponse>(model, schema, "SentimentResponse");
...
    const response = await translator.translate(request);
```

Pretty much everything else would be your application code that uses the "type definition" as input to carry out some business logic.

## How does it work?
If you analyze the code behind TypeChat, you'll realize that what happens under the hood is something downright simple.

Once you issue a "translate" command to TypeChat, below is what it does,
1. Figure out the `Model` to use, and its accompanying parameters (remember the ones we setup in `.env` file?)
2. Looks up the `type definition` of target output based on the schema configured
3. Issues a `completion` request to LLM with the following prompt

    ````plaintext
    You are a service that translates user requests into JSON objects of type "SentimentResponse" according to the following TypeScript definitions:

    // The following is a schema definition for determining the sentiment of a some user input.
    ```
    export interface SentimentResponse {
        sentiment: "negative" | "neutral" | "positive";  // The sentiment of the text
    }
    ```
    The following is a user request:
    """
    I feel so good today!
    """
    The following is the user request translated into a JSON object with 2 spaces of indentation and no properties with the value undefined:
    ````

    Let us assume the Model returned back a response like this:
    ````
    Here is the user request translated into a JSON object of type "SentimentResponse" with 2 spaces of indentation and no properties with the value undefined:
    ```json
    {
    "sentiment": "positive"
    }
    ```
    ````
4. If the Model returns back a valid JSON that complies to the Type definition, there isn't much to do - return the JSON object back to the caller
5. If the Model doesn't return a valid JSON, capture the validation error and issue another completion request (back to step #3) to the Model to attempt repair on the response, this time the prompt will look like below,

    ````plaintext
    You are a service that translates user requests into JSON objects of type "SentimentResponse" according to the following TypeScript definitions:
    ```
    // The following is a schema definition for determining the sentiment of a some user input.

    export interface SentimentResponse {
        sentiment: "negative" | "neutral" | "positive";  // The sentiment of the text
    }
    ```
    The following is a user request:
    """
    I feel so good today!
    """
    The following is the user request translated into a JSON object with 2 spaces of indentation and no properties with the value undefined:
    Here is the user request translated into a JSON object of type "SentimentResponse" with 2 spaces of indentation and no properties with the value undefined:
    ```json
    {
    "sentiment": "positive"
    }
    ```

    The JSON object is invalid for the following reason:
    """
    Response is not JSON:
    Here is the user request translated into a JSON object of type "SentimentResponse" with 2 spaces of indentation and no properties with the value undefined:
    ```json
    {
    "sentiment": "positive"
    }
    ```
    """
    The following is a revised JSON object:
    ````
    > Notice how the prior response has been embedded here to guide the Model to provide a better response.
    
    Lets assume the response now is as below,

    ```json
    {
        "sentiment": "positive"
    }
    ```
6. If the repair attempt also failed, return back a failure - repair is attempted only once

> Do note that if you issue these same prompts to Web version of ChatGPT, you might get slightly varying results

## Economy of using TypeChat
While we covered the uses of TypeChat and how easy it makes it to integrate natural language as one of the input means for systems that are already available, what could be the flipside of using it?

Lets start with cost - which should be one of the major factors to consider (especially in this case).

Even though TypeChat itself is Free & Open Source, it in itself cannot be used as a standalone component. You'll need a subscription to consume the Models. Today this is limited to OpenAI and Azure OpenAI offerings, but any LLMs that might get integrated with TypeChat isn't going to be available for free anytime soon!

First, let's grasp the pricing model. Both OpenAI and Azure offer the same pricing for the GPT-3.5 Turbo model, which is as follows.

| Model     | Input     | Output    |
|-----------|-----------|-----------|
| 4K context| $0.0015   | $0.002    |
| 8K context| $0.003    | $0.004    |

The charges are per 1,000 tokens consumed through Requests and Responses.

Per [OpenAI Pricing Docs](https://openai.com/pricing#language-models):
> You can think of tokens as pieces of words, where 1,000 tokens is about 750 words

While that might seem very generous, lets do some quick math to see how much we would end up spending.

Going by the example we saw earlier, and estimating the number of tokens on the prompt using [https://www.prompttokencounter.com/](https://www.prompttokencounter.com/), we would be consuming **~550 tokens** for that example (with close to a 50:50 split for input and output).

### Cost Per interaction:

| Model     | Count     | Pricing    | Cost | 
|-----------|-----------|-----------|-------|
| Average tokens consumed per interaction| 550   | -    | - |
| Input tokens| 275    | $0.0015/1K tokens    | $0.0004125 |
| Output tokens| 275    | $0.0020/1K tokens    | $0.00055 |
|   |     | **Total Cost**   | **$0.0009625** |

Now extrapolating this to an average of 10,000 interactions per day, we would end up running into a total cost of <u> **$288.75** </u> per month.

"Moreover, if you employ it in an application with a substantial user base and significant usage, your interactions and, consequently, costs will surpass what's outlined above.

## Testing in Local
OK, with this being so expensive is there a way to take this for a test drive without breaking the bank? The answer is, yes there is!

When I first made a few API calls to my ChatGPT instance, I soon realized that even for playing around with TypeChat and building a Demo/Proof-Of-Concept, accessing the real OpenAI/Azure APIs isn't going to be a viable option.

I was contemplating mocking OpenAI's API specifications into a locally built API that will be pointed to from TypeChat. While the idea sounded promising, it will be too much work on my part. That is when I learnt about [MockGPT](https://mockgpt.wiremock.io/), which is exactly what I had in mind.

MockGPT grants us access to a mock API that can respond with messages we're interested in, without requiring sign-up or imposing limitations. Since this API closely mirrors OpenAI's specifications, all we need to do is point to the URL and API Key provided by MockGPT.

> If you want to have better control over what responses are to be provided over what scenarios, you can create an account and setup your own API.

## Alternatives
TypeChat is not the only offering in this space, we actually have a lot of such libraries/tools which does similar job.

The ones that I am aware of are as below,
1. [LLMParser](https://www.llmparser.com/)
2. [LangChain](https://langchain.com/)
3. [Google Vertex AI](https://cloud.google.com/vertex-ai/docs/generative-ai/text/text-prompts#format-extracted-text)

Tools like LLMParser and LangChain are similar to TypeChat, where access to a separate instance of LLM is required, Google Vertex AI is a managed offering neatly packaging it all.

Since, LLMParser and LangChain are very similar to TypeChat, their operating cost would work out to be exactly same or very much aligned with what would be the cost when using TypeChat.

VertexAI on the other hand is priced a little differently - where they charge for every 1,000 characters in Input/Output response. Google charges $0.0005 per 1,000 characters. While this might seem very low compared to OpenAI's $0.002/1K token. Do note that OpenAI charges per 1,000 tokens while Google charges per 1,000 characters.

Despite this distinction, Google may be marginally less expensive than OpenAI, but the difference might not be substantial in the end.

## Conclusion
While tools like TypeChat does solve some key problems and ease the integration aspects when it comes to dealing with Natural Langugage inputs, it does come with a huge price tag. Like it always happens with Software/Engineering Suggestions - the answer to whether you  should be using this to solve your business problems is - "_it depends_". 

If the benefits you gain outweigh the costs of accessing these LLMs, why not leverage and make the most of them!

Happy Hacking!