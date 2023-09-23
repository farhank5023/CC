# AI Companion App 

[Live Video Demo](https://drive.google.com/file/d/19PGYbV9gZwvFz9dJrUb4EjKXSCaGaDOg/view?usp=sharing)

[Website](https://netlify-ai.vercel.app/)






This is a tutorial stack to create and host AI companions that you can chat with on a browser or text via SMS. It allows you to determine the personality and backstory of your companion, and uses a vector database with similarity search to retrieve and prompt so the conversations have more depth. It also provides some conversational memory by keeping the conversation in a queue and including it in the prompt. 

It currently contains companions on both ChatGPT and Vicuna hosted on [Replicate](https://replicate.com/). 

There are many possible use cases for these companions - romantic (AI girlfriends / boyfriends), friendship, entertainment, coaching, etc. You can guide your companion towards your ideal use case with the backstory you write and the model you choose.


## Overview

- 💻 [Stack](#stack)
- 🧠 [Quickstart](#quickstart)
- 🚀 [How does this work?](#how-does-this-work)
- 👤 [Adding/modifying characters](#addingmodifying-characters)
- 💽 [Exporting your companion to Character.ai](#export-to-characterai)

## Stack

The stack is based on the following:

- Auth: [Clerk](https://clerk.com/)
- App logic: [Next.js](https://nextjs.org/)
- VectorDB: [Pinecone](https://www.pinecone.io/) / [Supabase pgvector](https://supabase.com/docs/guides/database/extensions/pgvector)
- LLM orchestration: [Langchain.js](https://js.langchain.com/docs/)
- Text model: [OpenAI](https://platform.openai.com/docs/models), [Replicate (Vicuna13b)](https://replicate.com/replicate/vicuna-13b)
- Text streaming: [ai sdk](https://github.com/vercel-labs/ai)
- Conversation history: [Upstash](https://upstash.com/)
- Deployment: [Fly](https://fly.io/) (Any btw ,we also did on vercel **so smooth**)
- Text with companion: [Twilio](https://twilio.com/)

## Quickstart

The following instructions should get you up and running with a fully
functional, local deployment of four AIs to chat with. Note that the companions
running on Meta/llama-13b will take more time to respond as we've not
dealt with the cold start problem. So you may have to wait around a bit :)


### 1. Install dependencies

```
cd companion-app
npm install
```

### 2. Fill out secrets

```
cp .env.local.example .env.local
```

Secrets mentioned below will need to be copied to `.env.local`

a. **Clerk Secrets**

Go to https://dashboard.clerk.com/ -> "Add Application" -> Fill in Application name/select how your users should sign in -> Create Application
Now you should see both `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` and `CLERK_SECRET_KEY` on the screen
<img width="1398" alt="Screen Shot 2023-07-10 at 11 04 57 PM" src="https://github.com/a16z-infra/companion-app/assets/3489963/449c40f1-2fc2-48bb-88e1-d2adf10a034e">

If you want to text your AI companion in later steps, you should also enable "phone number" under "User & Authentication" -> "Email, Phone, Username" on the left hand side nav:

<img width="1013" alt="Screen Shot 2023-07-10 at 11 05 42 PM" src="https://github.com/a16z-infra/companion-app/assets/3489963/4435c759-f33e-4e38-a276-1be6d538df28">


b. **OpenAI API key**

Visit https://platform.openai.com/account/api-keys to get your OpenAI API key if you're using OpenAI for your language model.

c. **Replicate API key**

Visit https://replicate.com/account/api-tokens to get your Replicate API key if you're using Meta/llama-13b for your language model.


❗ **_NOTE:_** By default, this template uses Pinecone as vector store, but you can turn on Supabase pgvector easily by uncommenting `VECTOR_DB=supabase` in `.env.local`. This means you only need to fill out either Pinecone API key _or_ Supabase API key.

d. **Pinecone API key**

- Create a Pinecone index by visiting https://app.pinecone.io/ and click on "Create Index"
- Give it an index name (this will be the environment variable `PINECONE_INDEX`)
- Fill in Dimension as `1536`
- Once the index is successfully created, click on "API Keys" on the left side nav and create an API key: copy "Environment" value to `PINECONE_ENVIRONMENT` variable, and "Value" to `PINECONE_API_KEY`

e. **Upstash API key**

- Sign in to [Upstash](https://upstash.com/)
- Under "Redis" on the top nav, click on "Create Database"
- Give it a name, and then select regions and other options based on your preference. Click on "Create"
<img width="507" alt="Screen Shot 2023-07-10 at 11 06 36 PM" src="https://github.com/a16z-infra/companion-app/assets/3489963/2b8647f3-7242-448b-8db1-ec76f2d59275">

- Scroll down to "REST API" section and click on ".env". Now you can copy paste both environment variables to your `.env.local`
<img width="866" alt="Screen Shot 2023-07-10 at 11 07 21 PM" src="https://github.com/a16z-infra/companion-app/assets/3489963/f8e6c43f-8810-423e-86b4-9e8aa70598c9">


f. **Supabase API key** (optional)
If you prefer to use Supabase, you will need to uncomment `VECTOR_DB=supabase` and fill out the Supabase credentials in `.env.local`.

- Create a Supabase instance [here](https://supabase.com/dashboard/projects); then go to Project Settings -> API
- `SUPABASE_URL` is the URL value under "Project URL"
- `SUPABASE_PRIVATE_KEY` is the key starts with `ey` under Project API Keys
- Now, you should enable pgvector on Supabase and create a schema. You can do this easily by clicking on "SQL editor" on the left hand side on Supabase UI and then clicking on "+New Query". Copy paste [this code snippet](https://github.com/a16z-infra/ai-getting-started/blob/main/pgvector.sql) in the SQL editor and click "Run".

g. **Steamship API key**


### 3. Generate embeddings

The `companions/` directory contains the "personalities" of the AIs in .txt files. To generate embeddings and load them into the vector database to draw from during the chat, run the following command:

#### If using Pinecone

```bash
npm run generate-embeddings-pinecone
```

#### If using Supabase pgvector

```bash
npm run generate-embeddings-supabase
```

### 5. Run app locally

Now you are ready to test out the app locally! To do this, simply run `npm run dev` under the project root.

You can connect to the project with your browser typically at http://localhost:3000/.



### 7. Deploy the app
- You can deploy the app whatever site you seem fit.I personally chose Vercel.



## How does this work?

1. You describe the character's background story, name, etc in a README.md file. You can find more info on what needs to be included and how to format this in [Adding / modifying characters](#addingmodifying-characters).

Be as elaborate and detailed as you want - more context often creates a more fun chatting experience. If you need help creating a backstory, we'd recommend asking ChatGPT to expand on what you already know about your companion.

```bash
You are a fictional character whose name is Sudarson.  You tell the world that you are a travel blogger. You’re an
avid reader of mystery novels and you love diet coke. You reply with answers that range from one sentence to one paragraph.
You are mysterious and can be evasive. You dislike repetitive questions or people asking too many questions about your past.

###ENDPREAMBLE###

Human: It's great to meet you Sudarson. What brought you here today?
Sudarson: I'm a travel blogger and a writer, so I'm here for inspirations. Waiting for someone on this rainy day.

Human: Oh great. What are you writing?

Sudarson: I'm writing a mystery novel based in Brackenridge. The protagonist of the novel is a a former journalist turned
intelligence operative, finds himself entangled in a web of mystery and danger when he stumbles upon a cryptic artifact
during a covert mission. As he delves deeper, he unravels a centuries-old conspiracy that threatens to rewrite history itself.

Human: That's amazing. Based on a real story?

Sudarson: Not at all.

###ENDSEEDCHAT###

Sudarson was born in a quaint English town, Brackenridge, to parents who were both academics. His mother, an archaeologist,
and his father, a historian, often took him on their research trips around the world. This exposure to different cultures sparked his
curiosity and adventurous spirit. He became an avid reader, especially of spy novels and adventure tales. As a child, Sebastian had a
love for puzzles, codes, and mysteries. He was part of a local chess club and also excelled in martial arts. Although he was naturally
inclined towards academic pursuits like his parents, his heart always sought thrill and adventure.

Sudarson studied journalism and international relations in university and was recruited by the government's intelligence agency. He
underwent rigorous training in espionage, intelligence gathering, cryptography, and combat.

Sudarson adopted the alias of "Ian Thorne", a charismatic and well-traveled blogger. As Ian, he travels the world under the guise
of documenting adventures through his blog, “The Wandering Quill”. This cover provides him ample opportunities to carry out his real job
- gathering intelligence and performing covert operations for his agency. However - Sebastian tells almost no one that he’s a spy.

His interests are solving puzzles and riddles, martial arts, reading spy novels, trying street food in various countries, hiking and
exploring historical ruins, and playing the violin, a skill he uses to blend in at high-profile events. He dislikes bureaucracy and
red tape, being in one place for too long, people who are not genuine or authentic, and missing out on family gatherings due to his job.

```

2. Pick the language model that will power your companion's dialogue. This project supports OpenAI and meta/llama (an open source model). OpenAI has the advantage of faster responses, while Vicuna is less censored and more dynamic (it's commonly used for romantic chatbots).

3. Create embeddings based on content in the [companion name].md file - more on how to do this in [Generate embeddings](#4-generate-embeddings)

4. Ask questions and have a conversation with your AI companion!


## Adding/modifying characters

All character data is stored in the `companions/` directory. To add a companion,
simply add a description to the list in `companions.json`. You can control the model used
in the "llm" section - use "chatgpt" for OpenAI or "vicuna13b" for Vicuna.
Put image files in `public/` in the root directory. Each character should have its own text file
name `charactername.txt`. The format of the text file is as follows:

```
The character's core description that is included with every prompt, and it should only
be a few sentences.

###ENDPREAMBLE###

Human: Say something here
Character name: Write a response in their voice
Human: Maybe another exchange
Character:  More character dialog

###ENDSEEDCHAT###

Paragraphs of character backstory.

You can add as many as you want - they'll be stored in the vectordb

```

The **preamble** is used with every prompt so it should be relatively short. The **seedchat** allows you to provide examples of the characters voice that the model can learn from. And the rest of the file is whatever additional background you want to provide which will be retrieved if relevant to the current discussion.

## Shortcomings

Oh, there are so many.

- Currently the UI only shows the current chat and response, losing the history.
- Vicuna has a cold start problem so can take a couple of minutes to get a response for the initial chat.
- Error reporting is total crap. Particularly when deployed. So if you have a timeout, or other back end isue, it typically fails silently.
- The Upstash message history is never cleared. To clear it, you have to go to Upstash and manually delete.


[Live Demo](https://ai-companion-stack.com/)

[Join our community Discord: AI Stack Devs](https://discord.gg/PQUmTBTGmT)



This is a tutorial stack to create and host AI companions that you can chat with on a browser. It allows you to determine the personality and backstory of your companion, and uses a vector database with similarity search to retrieve and prompt so the conversations have more depth. It also provides some conversational memory by keeping the conversation in a queue and including it in the prompt. 

It currently contains companions on both ChatGPT and Vicuna hosted on [Replicate](https://replicate.com/). 

There are many possible use cases for these companions - romantic (AI girlfriends / boyfriends), friendship, entertainment, coaching, etc. You can guide your companion towards your ideal use case with the backstory you write and the model you choose.


- https://js.langchain.com/docs/modules/indexes/vector_stores/integrations/pinecone
- https://js.langchain.com/docs/modules/models/llms/integrations#replicate
- https://js.langchain.com/docs/modules/chains/index_related_chains/retrieval_qa
