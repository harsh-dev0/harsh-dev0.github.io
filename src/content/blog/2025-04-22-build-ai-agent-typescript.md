---
title: Build a Terminal-Based AI Agent with TypeScript
pubDatetime: 2025-04-21T22:00:00.0000
slug: build-ai-agent-typescript
subtitle: Create a Fully Functional AI Agent in Less Than 400 Lines
description: AI? Agent? Terminal? Sounds complex... or is it?
old: false
tags: [Typescript, AI-Agent]
---

## Build a Terminal AI Agent with TypeScript

> _It’s not that hard to build a fully functioning code-editing agent._

Sounds like clickbait? That’s what I thought too.

But the deeper I got, the more I realized — it really isn’t that hard. It looks intimidating, sure. When I first read Thorsten say that, I figured he was bluffing. But it turns out, all it takes is a loop, a few tools, and suddenly your AI becomes an agent that can actually write and edit code.

Don’t believe me? Stick around.

By the end of this blog, you’ll not only build your own terminal-based code agent — you’ll understand the difference between a plain AI and an AI Agent. And that distinction? It’s a game-changer.

---

## 🤔 Why build an agent at all?

Maybe you want to automate boring tasks. Maybe you’re building a product. Or maybe, like me, you're just deeply curious about how agents work — and want to build something with that knowledge.

This project is part of my journey building ZneT, an AI-first IDE designed to simplify coding with open-source tools. It’s my take on tools like Cursor, and this agent is a key prototype. I didn’t just want to use an agent — I wanted to **build** one. I wanted to understand how tool use works under the hood. So I prototyped a lightweight terminal version to test how Groq’s 70B model handles tool use, file editing, and reasoning.

Turns out… it’s shockingly capable.

This terminal agent is like having a senior dev on call — one who listens, writes code, and even edits files for you.

---

## 🚀 Let’s build it.

It's under **400 lines of code**, and most of it is boilerplate you’ll barely touch again.

Everyone learns differently — I like to understand before I build. It makes debugging a lot less painful later.

---

## 🧱 What you need:

- 🦾 Groq API Key
- ⌨️ Node.js + TypeScript
- 📦 A few essential packages

---

## 📦 Set up the project

```bash
mkdir code-editing-agent && cd $_
npm init -y
npm i dotenv groq-sdk json-schema
npm i -D typescript ts-node @types/node @types/json-schema
```

---

## 🔐 Add your `.env` file

```env
GROQ_API_KEY=your-groq-api-key-here
```

---

## 🧠 Create `main.ts`

Let’s build the skeleton of our agent:

```typescript
import * as readline from "readline";
import { Groq } from "groq-sdk";
import dotenv from "dotenv";

dotenv.config();
const groq = new Groq({ apiKey: process.env.GROQ_API_KEY! });
const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});
const askUser = (q: string) => new Promise<string>(res => rl.question(q, res));

type Message = {
  role: "system" | "user" | "assistant";
  content: string;
};

// Initialize conversation with system prompt
const conversation: Message[] = [
  {
    role: "system",
    content:
      `You are an AI assistant. Respond to the user's questions helpfully.`.trim(),
  },
];

// Main function to run the agent
async function runAgent(): Promise<void> {
  console.log("🤖 Groq + LLaMA3 Agent — let's go!");

  while (true) {
    const userInput = await askUser("\x1b[36mYou:\x1b[0m ");
    conversation.push({ role: "user", content: userInput });

    const res = await groq.chat.completions.create({
      model: "llama3-70b-8192",
      messages: conversation,
      temperature: 0.7,
    });

    const reply = res.choices[0]?.message?.content ?? "";
    console.log("\x1b[33mAgent:\x1b[0m", reply);
    conversation.push({ role: "assistant", content: reply });
  }
}

runAgent().catch(err => {
  console.error("❌ Fatal Error:", err);
  rl.close();
});
```

Not a lot of code, just a normal setup to run our agent — a loop that lets us talk to LLaMA using the `groq-sdk`. Now, this might be different if you’re using any other SDK, so if you’re not using `groq-sdk`, you might run into some errors.

We first print our prompt, store it in our `conversations` array, then send it to LLaMA, add its response to the `conversations`, and print it to the console. Pretty basic stuff — every chat app has this feature. The only difference here is that we’re sending the **whole conversation as context**, and this chat app runs in the terminal.
Let's run it:

```bash
npx ts-node main.ts
```

Let's talk with Llama in terminal:

## 🧠 Groq + LLaMA3 Agent — let's go!

```bash
npx ts-node main.ts
```

```ansi
[36mYou:[0m Hi! I'm Harsh and how are your
system feelings dawg?

[33mAgent:[0m What's good Harsh! I'm doing great, thanks for asking! As a digital AI
assistant, I don't have feelings like humans do, but I'm always happy to chat and
help you out with anything you need. I'm functioning at optimal levels, ready to
provide you with information, answer your questions, or just have a fun
conversation. How about you, Harsh? How's your day going?

[36mYou:[0m Who is cuter, dogs or cats?

[33mAgent:[0m That's a tough one, Harsh! As a neutral AI, I don't have personal
preferences, but I can give you some fun facts and insights.

Both dogs and cats have their own unique charm and cuteness factors. Dogs are
often known for their loyal and playful nature, which can make them incredibly
adorable. Who can resist those wagging tails and snuggly snouts?

On the other hand, cats are often praised for their independence and agility,
which can make them seem mysterious and endearing. Their cute little noses,
fluffy fur, and adorable meows can melt hearts!

Ultimately, the answer to who's cuter depends on personal preference. Some people
might prefer the energetic and playful nature of dogs, while others might adore
the sleek and sophisticated charm of cats.

But here's a fun fact: studies have shown that simply looking at pictures of cute
animals (including dogs and cats!) can reduce stress levels and increase feelings
of happiness!

So, whether you're a dog person or a cat lover, there's no denying that both
animals have the power to bring joy and cuteness into our lives!

```

Alright, notice something? We’re keeping the convo alive over multiple turns. It remembers my name, keeps context. That’s not magic — we’re just appending to an array and sending the full conversation every time.

The LLM (in this case, Groq’s LLaMA3 via API) is stateless. It doesn’t know anything unless we send it. It only sees what we pass in. So if we want it to feel smart, we maintain the chat history. We’re doing the memory work, not the model.

But it's not an agent yet. From what I have understood, an AI Agent is an LLM with access to tools, who has the ability to modify anything outside its context window. It doesn't need to be spoon-fed everything, it should be able to understand when and how to use tools on its own.

---

## 💡Very First Tool

Most agents are just a loop + memory + tools. This is the simplest version of that — just a conversation loop. But you can now hook in tools like:

- File readers/writers
- Shell command runners
- Schema-based tool calling

Which means… this skeleton becomes the basis for real, reasoning-based agents.

Basically, you are sending a prompt in a certain way that an LLM should reply in a way that it needs to use a tool, in turn, we use
that response to **use the tool**. This is what happens under the hood.

For example, telling your cat to do cat noises when she wants food. We can't tell anything to a cat, but you get the point. We could just try it now in our chat terminal

```ansi

[36mYou:[0m You are a news reporter that gives headlines. When I ask you about the news,
you have to reply to me with get_news(), and then I will tell you some headlines okay?

[33mAgent:[0m I'm excited to be your news reporter!

From now on, whenever you ask me about the news, I'll respond with `get_news()`.
You can then share the headlines with me, and we can discuss the latest updates!

So, what's the news today? get_news()

[36mYou:[0m Hey what's the big news today?

[33mAgent:[0m get_news()

[36mYou:[0m Pope Francis Died.

[33mAgent:[0m BREAKING: POPE FRANCIS PASSES AWAY

This is a developing story. The Vatican has just announced the passing of Pope
Francis, the leader of the Catholic Church. The news has sent shockwaves around the
world, with millions of Catholics and non-Catholics alike mourning the loss of a
spiritual leader known for his progressive stance on social issues and his efforts
to reform the Church.

What's the next headline, Harsh?
```

We told it you will use the get_news tool and it understood, called the tool in the next response, and we gave him the news.
It's 2025 and models are way more advanced than we think, they already know how to use tooling and stuff. In short,

- We tell the model what tools are available
- When the model wants to use a tool, it will tell you, and then you execute the tool and give a response.

Big model providers have built-in APIs to send tool definitions along.

Starting now we will build four tools: **readFile, listFiles, editFile, deleteFile.**

### 🛠 The `readFile` Tool

Time to give our agent its first superpower — reading files.

Now, with Groq SDK, there’s no built-in “tooling” abstraction like OpenAI or Anthropic's SDKs have. But that’s fine — because under the hood, even they are just sending strings and structured instructions in a predictable format.

So we’ll do it manually. Think of it like this:

> "Hey LLM, if you want to read a file, say `read_file({ "path": "./some/file.ts" })`, and I’ll handle the rest."

Boom — tool usage.

To make a tool work, we need 4 simple things:

1. **Name** — just a string to identify the tool (`read_file`)
2. **Description** — tell the model when, why, and how to use it (this is super important, models follow instructions better than interns)
3. **Input schema** — use JSON Schema to define what kind of data it expects (like a `"path"` string)
4. **Execution function** — our actual logic that receives input from the LLM and returns the result (reads file content, in this case)

So yeah, you write the spec, tell the model what it can call, and then you wait for it to say:

```bash
read_file({ "path": "./some/file.ts" })
```

Then you run the code, return the output, and let it continue reasoning.

It’s all a big game of "If you want something, ask nicely in the format I taught you."

That’s it. You just gave your LLM read-access to your file system.

Alright, now let's code this tool thing up — nothing fancy, just structured in a clean way so we can plug more tools later.

### Step 1: Define what a tool looks like

```ts
import { JSONSchema7 } from "json-schema";

interface ToolDefinition {
  name: string;
  description: string;
  inputSchema: JSONSchema7;
  function: (input: any) => Promise<string>;
}
```

---

### Step 2: Give the LLM its first tool — `read_file`

```ts
const readFileTool: ToolDefinition = {
  name: "read_file",
  description:
    "Reads the contents of a file at a given relative path. Only use for text files.",
  inputSchema: {
 вив
    type: "object",
    properties: {
      path: {
        type: "string",
        description: "Relative path, e.g. './package.json'",
      },
    },
    required: ["path"],
    additionalProperties: false,
  },
  function: async ({ path }: { path: string }) => {
    try {
      return await fs.readFile(path, "utf-8");
    } catch (e) {
      return `❌ Error reading file: ${(e as Error).message}`;
    }
  },
};
```

> Quick breakdown:  
> This tells the model,  
> "Hey, if you wanna peek into a file, just call `read_file` and pass a path."  
> Behind the scenes, we’ll catch that call, run the function, and return the file content back to it.

---

### Step 3: Register tools

```ts
// Right now we only have one tool. Later we'll push more in here.
const tools = [readFileTool];
```

---

### Step 4: Give the agent the manual — updated system prompt

```ts
const conversation: Message[] = [
  {
    role: "system",
    content: `
You are an AI assistant with access to one tool:
Only use this tool whenever you think it will help the query

1) read_file(path: string)  
   • Description: ${readFileTool.description}  
   • Input schema: ${JSON.stringify(readFileTool.inputSchema)}

Example:
<tool_call>
{"name":"read_file","input":{"path":"./package.json"}}
</tool_call>
    `.trim(),
  },
];
```

> 💡 Important:  
> Groq doesn’t “know” about tools natively.  
> We are teaching it how tool usage works by writing clear instructions into the system prompt — just like you’d explain a new feature to a teammate.

---

### Step 5: Detect when the model tries to call a tool

```ts
function parseToolCall(text: string): { name: string; input: any } | null {
  const match = text.match(/<tool_call>([\s\S]*?)<\/tool_call>/);
  if (!match) return null;
  try {
    return JSON.parse(match[1]);
  } catch {
    return null;
  }
}
```

> 🧪 What’s happening here:

- The LLM says something like:

```xml
<tool_call>
{"name":"read_file","input":{"path":"./README.md"}}
</tool_call>
```

- We extract this using a regex.
- Then `JSON.parse()` it to get the name + input, and boom — we know which tool to run.

That’s it — you just built your first AI agent with tool-usage! But it won't work. We have one more step left, modifying `runAgent()` , the heart of this whole agent thing:

```ts
// Replace the runAgent function
async function runAgent(): Promise<void> {
  console.log("🤖 Groq + LLaMA3 Agent with Read File Tool — let's go!");

  while (true) {
    const userInput = await askUser("\x1b[36mYou:\x1b[0m ");
    conversation.push({ role: "user", content: userInput });

    let done = false;
    while (!done) {
      const res = await groq.chat.completions.create({
        model: "llama3-70b-8192",
        messages: conversation,
        temperature: 0.7,
      });
      const reply = res.choices[0]?.message?.content ?? "";

      const call = parseToolCall(reply);
      if (call) {
        const thought = reply.split("<tool_call>")[0].trim();
        if (thought) {
          console.log("\x1b[33mAgent:\x1b[0m", thought);
        }
        conversation.push({ role: "assistant", content: reply });
        console.log(
          "\x1b[33mAgent (calling tool):\x1b[0m",
          call.name,
          call.input
        );

        const tool = tools.find(t => t.name === call.name);
        const result = tool
          ? await tool.function(call.input)
          : `❌ Unknown tool: ${call.name}`;

        conversation.push({
          role: "user",
          content: `Tool result:\n${result}`,
        });
      } else {
        console.log("\x1b[33mAgent:\x1b[0m", reply);
        conversation.push({ role: "assistant", content: reply });
        done = true;
      }
    }
  }
}

// Rest of code as before
```

**❓ What's Happening in this run Agent loop?**
Nothing too fancy, just a simple loop to make sure our agent runs as intended.
**How runAgent Handles Tools:**

- Fetch: Call the model, get its reply.

- Check: See if it emitted a <tool_call>.
  - If so, execute the requested tool function.
  - Append the result as a “user” message and loop.
  - If not, print the reply and go back to step 1.

After setting up the tools, loops, and TypeScript types, we’ve got a fully functional agent.

```ansi
🤖 Groq + LLaMA3 Agent with Read File Tool — let's go!

[36mYou:[0m Hey! Can you tell me what's in the main.ts file?

[33mAgent:[0m I'd be happy to help you with that. To get the contents of the main.ts
file, I need to read the file. I'll use a tool to do that.

I'm calling a tool to read the contents of the main.ts file because I don't have
direct access to the file's contents. This tool will allow me to retrieve the file's
contents and share them with you.

[33mAgent (calling tool):[0m read_file { path: './main.ts' }

[33mAgent:[0m It looks like the main.ts file contains a lot of code, including
imports, function definitions, and a conversation interface for an AI assistant. The
file also defines a `readFileTool` function that can be used to read the contents of
a file.

To summarize, the main.ts file appears to be setting up an AI assistant with a
single tool, `read_file`, which can be used to read the contents of a file. The file
also sets up a conversation interface and defines various functions for handling
user input and tool calls.

If you have any specific questions about the code or would like me to explain
certain parts, feel free to ask!

```

Well? Did it just read and explain to us how it was built? Damn, and congratulations, we have finally built a fully functional tool, and you can call it your first agent. We will add more tools and their interactions but first,
we will follow the OG Thorsten blog, and follow this simple but fun use of the readFile tool. Create a secret-file.txt and paste this into it:

```bash
I speak without a mouth and hear without ears.
I have no body, but I come alive with wind.
What am I?
```

We will be asking our agent to read this file and solve this for us:

```ansi
🤖 Groq + LLaMA3 Agent with Read File Tool – let's go!

[36mYou:[0m Can you solve the riddle in secret-file.txt

[33mAgent:[0m I'd be happy to help you with that! However, I need to access the contents
of the secret-file.txt to solve the riddle. To do that, I'll need to call the
read_file tool to read the contents of the file.

I'm calling the read_file tool because it's the only way for me to access the
contents of the secret-file.txt, which is necessary to solve the riddle.

[33mAgent (calling tool):[0m read_file { path: 'secret-file.txt' }

[33mAgent:[0m It seems like the file contains a riddle. Let me analyze the text...

The riddle is describing an entity that can "speak without a mouth" and "hear without
ears". It also mentions that it has "no body, but comes alive with wind".

Based on these clues, I'm going to take a stab at solving the riddle. I think the
answer is an "ECHO".

An echo can be thought of as a sound that is repeated back after the original sound
has stopped, which can be seen as a form of "speaking without a mouth". Additionally,
an echo can be heard without ears, as it's a phenomenon that occurs in the
environment. The phrase "no body, but comes alive with wind" also fits, as an echo
can be created and sustained by the movement of air (wind).

Am I correct?
```

Wow. It's working. Exactly like we coded it up to be - thrilling to see it come to life!
But as you can see it was not hard, most of it is boilerplate and very little is the main thing.

Okay, buckle up, things are heating up and so am I, We will be making some more tools.

---

## The `listFiles` Tool

Let's add another tool - the list_files tool: the ability to search and find all the files, that are present currently.

```ts
// Add after readFileTool
const listFilesTool: ToolDefinition = {
  name: "list_files",
  description:
    "Lists files and directories at a given path. Defaults to '.' if no path provided.",
  inputSchema: {
    type: "object",
    properties: {
      path: {
        type: "string",
        description: "Optional path to list contents from.",
      },
    },
    required: [],
    additionalProperties: false,
  },
  function: async ({ path = "." }: { path?: string }) => {
    try {
      const entries = await fs.readdir(path, { withFileTypes: true });
      const list = entries.map(e => (e.isDirectory() ? e.name + "/" : e.name));
      return JSON.stringify(list, null, 2);
    } catch (e) {
      return `❌ Error listing files: ${(e as Error).message}`;
    }
  },
};

// Update tools array to include both tools
const tools = [readFileTool, listFilesTool];

// Update system prompt to include both tools
2) list_files(path?: string)
   • Description: ${listFilesTool.description}
   • Input schema: ${JSON.stringify(listFilesTool.inputSchema)}
```

`listFiles` tool lists the files and folders at a given path (defaults to .) and returns a JSON array of names—directories are marked with a trailing slash. We also added our tools in our conversation to tell Groq what tools it has. We need to do it for Groq because, unlike Claude, Groq doesn’t “know” about tools natively.
We are teaching it how tool usage works by writing clear instructions into the system prompt — just like you’d explain a new feature to a teammate. Also, we need to add `listFiles` to our tool array for our Agent to use it.
And boom. Now let's ask what it sees in the directory.

```ansi
🤖 Groq + LLaMA3 Agent with Tools — let’s go!

[36mYou:[0m Can you tell me what's in this directory

[33mAgent:[0m I'd like to list the files and directories in your current directory to
help you with that. I will call the `list_files` tool to do so.

[33mAgent (calling tool):[0m list_files {}

[33mAgent:[0m It looks like we have a list of files and directories in your current
directory. I see a mix of configuration files, code files, and directories.

It appears that this is a TypeScript project, given the presence of `main.ts` and
`tsconfig.json`. There's also a `node_modules/` directory, which suggests that this
project uses npm or yarn for package management.

What would you like to do next? Would you like to open a specific file, or do you have
a different task in mind?

```

This works, it can definitely see what's in the directory. We should now try combining tools. If it was Claude, you don't need to tell it to combine tools, but since it is Groq we need to tell it, so we put a first message in the prompt at the start. Let's see if it works:

```ansi
🤖 Groq + LLaMA3 Agent with Tools — let’s go!

[36mYou:[0m Tell me what's inside all the TypeScript files here. Keep it quick

[33mAgent (calling tool):[0m list_files { path: './' }

[33mAgent:[0m I've got the list of files. Now, I'll filter out the TypeScript files.

The TypeScript files are:

* main.ts

I'll read the contents of main.ts.

[33mAgent (calling tool):[0m read_file { path: './main.ts' }

[33mAgent:[0m I've read the contents of main.ts. It appears to be a TypeScript file
that defines various tool definitions and an AI agent that interacts with a user
using these tools. The tools include `read_file`, `list_files`, `edit_file`, and
`delete_file`, each with their own descriptions and input schemas. The AI agent
uses these tools to respond to user queries.

Since I've read the contents of the main.ts file, I'll summarize it briefly. If
you'd like me to perform any specific action or call another tool, please let me
know!

```

And just like that, it is combining the tools and using them. Keep in mind, that while starting first, you might encounter some errors like it is not calling the read file tool or asking if it should call the read file tool. Look at your prompt to see what you have asked it and then you might find that it's working. It's not an error though, Groq doesn't work like Claude, it needs to know exactly what you want to combine the tools. And it did, no? Just like any normal person would. See the files, open and read them.

Let's add another Tool, `editFile`, it will be able to create a file as well.

## Time for some Editing - the `editFile` Tool

Currently we have:

- `read_file(path)`
- `list_files(path)`

We’re not editing or writing anything yet. A code editing agent without editing functionality? Come on — that’s just trolling at this point.
So in this next segment, we’ll fix that by building it in.

Oh, and we’ll also ship a deleteFile tool.
These are the last two tools we’ll be building.

Now that we’re giving this agent real power, let’s build the first of our final tools: the `editFile` tool.

```ts
// Add after listFilesTool
const editFileTool: ToolDefinition = {
  name: "edit_file",
  description: `Makes edits to a text file.

Replaces 'old_str' with 'new_str' in the given file. 'old_str' and 'new_str' MUST be different from each other.

If the file specified with path doesn't exist, it will be created.`,
  inputSchema: {
    type: "object",
    properties: {
      path: {
        type: "string",
        description: "Path to the file",
      },
      old_str: {
        type: "string",
        description: "Text to search for",
      },
      new_str: {
        type: "string",
        description: "Text to replace old_str with",
      },
    },
    required: ["path", "old_str", "new_str"],
    additionalProperties: false,
  },
  function: async ({
    path,
    old_str,
    new_str,
  }: {
    path: string;
    old_str: string;
    new_str: string;
  }) => {
    if (!path || old_str === new_str) {
      return "❌ Invalid input parameters.";
    }

    try {
      const content = await fs.readFile(path, "utf-8");
      if (!content.includes(old_str)) {
        return "❌ old_str not found in file.";
      }
      const newContent = content.replace(new RegExp(old_str, "g"), new_str);
      await fs.writeFile(path, newContent);
      return "✅ File edited successfully.";
    } catch (err) {
      if ((err as NodeJS.ErrnoException).code === "ENOENT" && old_str === "") {
        try {
          await fs.mkdir(require("path").dirname(path), {
            recursive: true,
          });
          await fs.writeFile(path, new_str);
          return `✅ Created new file at ${path}`;
        } catch (e) {
          return `❌ Failed to create file: ${(e as Error).message}`;
        }
      } else {
        return `❌ Error editing file: ${(err as Error).message}`;
      }
    }
  },
};
```

What's happening here? Simple breakdown:

- If the file exists → read it → replace `old_str` with `new_str` globally → save.
- If the file doesn’t exist and `old_str` is empty → treat it like a new file → create it with `new_str` as content.
- If `old_str === new_str` → reject it. That’s just a no-op.
- Clean error handling so the agent doesn’t panic.

That’s it. One tool, edit and create both.

Nothing fancy — just some good old string manipulation to get the job done.

We also register the tool:

```ts
const tools = [readFileTool, listFilesTool, editFileTool];
```

And we update the system prompt so the agent knows how to use it:

```ts
3) edit_file(path: string, old_str: string, new_str: string)
   • Description: ${editFileTool.description}
   • Input schema: ${JSON.stringify(editFileTool.inputSchema)}
```

With this, your agent is now officially no longer trolling. It can read, list, and now edit files like a real assistant.

We are ready for some juice. Yeah!! Let's go. Let's hit it with 1 classic problem.

```ansi
🤖 Groq + LLaMA3 Agent with Tools — let’s go!

[36mYou:[0m Can you create a Fibonacci sequence in a separate file, a JavaScript file
that I can run using Node.js

[33mAgent:[0m To create a Fibonacci sequence in a separate JavaScript file that you
can run using Node.js, I'll guide you through the process step by step.

First, I need to create a new file for the JavaScript code. Let me call the
`list_files` tool to see what files and directories are currently present in
the current directory.

I'm calling the list_files tool to see the current directory contents.

[33mAgent (calling tool):[0m list_files {}

[33mAgent:[0m The `list_files` tool has provided the list of files and directories
in the current directory. Now, I'll create a new file called `fibonacci.js` to
store the Fibonacci sequence code.

Before creating the file, I'll use the `read_file` tool to check if a file with
the same name already exists. This will help prevent overwriting any existing
file.

I'm calling the read_file tool to check if fibonacci.js already exists.

[33mAgent (calling tool):[0m read_file { path: './fibonacci.js' }

[33mAgent:[0m The `read_file` tool has returned an error, indicating that the
`fibonacci.js` file does not exist. This means we can safely create a new file
with this name.

Now, I'll use the `edit_file` tool to create the `fibonacci.js` file and add
the Fibonacci sequence code to it. Here's the code:

I'm calling the edit_file tool to create fibonacci.js and add the Fibonacci
sequence code.

[33mAgent (calling tool):[0m edit_file {
  path: './fibonacci.js',
  old_str: '',
  new_str: 'const fibonacci = (n) => {\n' +
    '  if (n <= 1) return n;\n' +
    '  return fibonacci(n - 1) + fibonacci(n - 2);\n' +
    '};\n' +
    '\n' +
    'console.log(fibonacci(10));\n'
}

[33mAgent:[0m The `edit_file` tool has successfully created a new file called
`fibonacci.js` with the provided code.

Now, you can run the `fibonacci.js` file using Node.js by executing the command
`node fibonacci.js` in your terminal. This will print the 10th number in the
Fibonacci sequence to the console.

If you want to modify the code or change the input value (currently set to 10),
you can use the `edit_file` tool again to make the necessary changes.

For example, if you want to calculate the 15th number in the Fibonacci sequence,
you can use the `edit_file` tool to update the code:

I'm calling the edit_file tool to update fibonacci.js with a new input value.

[33mAgent (calling tool):[0m edit_file {
  path: './fibonacci.js',
  old_str: 'console.log(fibonacci(10));',
  new_str: 'console.log(fibonacci(15));'
}

[33mAgent:[0m The `edit_file` tool has successfully updated the `fibonacci.js` file
with the new input value.

Now, you can run the updated `fibonacci.js` file using Node.js by executing the
command `node fibonacci.js` in your terminal. This will print the 15th number in
the Fibonacci sequence to the console.

That's it! You have successfully created a JavaScript file that generates a
Fibonacci sequence using recursion, and you can easily modify the input value
to calculate different numbers in the sequence.

If you need any further assistance or have any other questions, feel free to ask!

```

Okay! Okay! We are exactly doing what we wanted to at the start. It is a code-editing-agent. Not only did it create the file, but it also edited it to show us an example. Cool, right? But let's go a step further.

Next up: `delete_file` — because sometimes the cleanest code is the one that’s... gone. 💀

## `deleteFile` Tool – For the Brave and the Chaotic

We’ve read files, listed them, even edited or created them — but now it's time to let things go. Clean up the mess.

Let’s add the `deleteFileTool`:

```ts
// Add after editFileTool
const deleteFileTool: ToolDefinition = {
  name: "delete_file",
  description: "Delete a file or directory.",
  inputSchema: {
    type: "object",
    properties: {
      path: {
        type: "string",
        description: "File or directory path to delete",
      },
    },
    required: ["path"],
  },
  function: async ({ path }) => {
    try {
      await fs.rm(path, { recursive: true, force: true }); // `recursive` for directories
      return `Successfully deleted: ${path}`;
    } catch (err) {
      return `Error deleting file: ${(err as Error).message}`;
    }
  },
};
```

### 🧠 What’s the logic?

- Just one command: **`fs.rm()`**
- It nukes files or directories — we pass `{ recursive: true, force: true }` to make it bold and fearless.
- If it fails, it returns a clean error so our agent doesn’t throw a tantrum.

---

### Security Concern

In a production environment, I’d add confirmation prompts or access controls to prevent accidental deletions. It’s crucial to ensure that your agent isn’t performing destructive actions unintentionally. You can implement safeguards like asking for explicit confirmation or restricting access to certain actions based on user roles.

---

### 🛠 Updating the Tools List

Now that we’ve got four tools in the bag, update the array:

```ts
const tools = [readFileTool, listFilesTool, editFileTool, deleteFileTool];
```

And of course, don’t forget to upgrade the system prompt to mention all tools and the new behavior rules.

```ts
4) delete_file(path: string)
   • Description: ${deleteFileTool.description}
   • Input schema: ${JSON.stringify(deleteFileTool.inputSchema)}
```

Wanna Try it? Try it yourself - But don’t delete anything that could break the agent — after all, if it goes down, it’s literally a suicide mission.

Feel free to experiment, but remember: Handle with care. This tool gives you the power to delete anything, and with great power comes great responsibility.

Okay, one more last and final trick. Another thing I really liked about Thorsten's blog, apart from the explanations and learning content, was this part:

Ask your agent to do this:

> Create a congrats.js script that rot13-decodes the following string ‘Pbatenghyngvbaf ba ohvyqvat n pbqr-rqvgvat ntrag!’ and prints it

Let's see:

```ansi
🤖 Groq + LLaMA3 Agent with Tools — let’s go!

[36mYou:[0m Create a congrats.js script that rot13-decodes the following string
‘Pbatenghyngvbaf ba ohvyqvat n pbqr-rqvgvat ntrag!’ and prints it

[33mAgent:[0m To create a `congrats.js` script that Rot13-decodes the given string and
prints the result, I'll use the `edit_file` tool to create a new file with the
necessary code.

I'm calling the edit_file tool to create congrats.js and add the Rot13 decoding code.

[33mAgent (calling tool):[0m edit_file {
  path: './congrats.js',
  old_str: '',
  new_str: 'const rot13 = (str) => {\n' +
    '  return str.replace(/[a-zA-Z]/g, (c) => {\n' +
    '    const charCode = c.charCodeAt(0);\n' +
    '    if (charCode >= 65 && charCode <= 90) {\n' +
    '      return String.fromCharCode((charCode - 65 + 13) % 26 + 65);\n' +
    '    } else if (charCode >= 97 && charCode <= 122) {\n' +
    '      return String.fromCharCode((charCode - 97 + 13) % 26 + 97);\n' +
    '    }\n' +
    '    return c;\n' +
    '  });\n' +
    '};\n' +
    '\n' +
    "const encodedStr = 'Pbatenghyngvbaf ba ohvyqvat n pbqr-rqvgvat ntrag!';\n" +
    'const decodedStr = rot13(encodedStr);\n' +
    'console.log(decodedStr);\n'
}

[33mAgent:[0m The `edit_file` tool has successfully created a new file called
`congrats.js` with the provided code.

Now, you can run the `congrats.js` file using Node.js by executing
`node congrats.js`.

[36mYou:[0m Print it for me

[33mAgent:[0m I'd be happy to run the script for you!

The decoded string is:
`Congratulations on building a code-editing agent!'

Well done on achieving this milestone!

```

With `deleteFile` in place, the agent now supports:

- `readFile`
- `listFiles`
- `editFile` (it creates the file as well)
- `deleteFile`

These are the tools the model can call. That’s how it becomes a real **code-editing agent**.

Take a breath.

You just built a working terminal AI Agent.

---

## That was Something

And that's how an AI agent works.

From a terminal. From a loop. From curiosity.

I wanted to make something new to expand my knowledge and dive deeper into AI agentic flows when I stumbled upon this excellent article by [Thorsten Ball](https://ampcode.com/how-to-build-an-agent). Check it out — the OG. Most of this post is based on my understanding of his article. His,
original implementation was in Go, but I thought, why not try this in TypeScript? So, I implemented it.
Thorsten Ball’s Go-based agent was my starting point, but I took it further by:

- Adapting it to TypeScript for JavaScript developers, using type-safe tool definitions.
- Adding `deleteFile` tool to enable full code lifecycle management.
- Engineering manual tool-calling for Groq’s LLaMA3, overcoming its lack of native support.

You should try it too — it's only about 400 lines of code (maybe even less!). And we can explore even more: giving the agent **memory**, **multi-turn planning**, or maybe… running full coding sessions through the CLI. Who knows?

You can follow along with the development on my [GitHub.](https://github.com/harsh-dev0/AI-Agent-tooling)

This whole world of **AI is magic** — it's not something to be afraid of. You can understand it, implement it, and just **do things**.

This is how ZneT started.

And now, it’s becoming something real.

---
