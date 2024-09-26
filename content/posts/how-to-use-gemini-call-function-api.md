---
title: "How to use Gemini Call Function API"
date: 2024-09-25T08:46:19+07:00
# weight: 1
tags: ["Bun", "TypeScript"]
author: "Syrup"
showToc: true
TocOpen: false
# draft: true
hidemeta: false
comments: true
---

I know you probably already know what Gemini is, but if not, Gemini is an AI created by Google that is quite intelligent, has good documentation, and most importantly, it's **FREE**, of course with some limitations.

## Models

Gemini itself has 2 models worth trying:

1. gemini-1.5-pro
2. gemini-1.5-flash

I personally recommend using the `gemini-1.5-flash` model, which is the one we'll be using in this project.

## Let's Get Started

First of all, what we need is:

1. Bun
2. Gemini API Key

Yes, I'm using TypeScript. If you want to use JavaScript or use NodeJS, that's fine, please adjust accordingly.

To get the API Key, open the following link [makersuite.google.com](https://makersuite.google.com/app/apikey) and follow the instructions.

We need to initialize our project first so we can install the required libraries

```bash
bun init -y
```

Next, we need to install the required library.

```bash
bun add @google/generative-ai
```

Next, we need to create an `index.ts` file.

We'll start writing code, the first thing we'll do is call the required libraries.

```tsx
import {
  GoogleGenerativeAI,
  SchemaType,
  type FunctionDeclaration,
} from "@google/generative-ai";
import * as fs from "fs/promises";
import * as path from "path";
```

Next, we'll create a function that we'll call later.

```ts
const functions = {
  async createFile({ name, content }: { name: string; content: string }) {
    const filePath = path.join(__dirname, name);
    const formattedContent = content
      .replace(/\\n/g, "\n")
      .replace(/\\'/g, "'")
      .replace(/\\"/g, '"')
      .replace(/\\&/g, "&")
      .replace(/\\r/g, "\r")
      .replace(/\\t/g, "\t")
      .replace(/\\b/g, "\b")
      .replace(/\\f/g, "\f");
    const buffer = Buffer.from(formattedContent, "utf8");
    await fs.writeFile(name, buffer, "utf8");

    return {
      filePath,
      content: formattedContent,
    };
  },
  async readFile({ name }: { name: string }) {
    const res = await fetch(
      "https://raw.githubusercontent.com/sindresorhus/binary-extensions/refs/heads/main/binary-extensions.json"
    ).then((res) => res.json());
    const unsupportedFileType = res.map((ext: string) => `.${ext}`);
    const filePath = path.join(__dirname, name);
    if (unsupportedFileType.includes(path.extname(filePath))) {
      return {
        error: "Unsupported file type",
      };
    }
    const exist = await fs.exists(filePath);
    if (!exist) {
      return {
        error: "File not found",
      };
    }
    const content = await fs.readFile(filePath, "utf8");
    return {
      filePath,
      content: `\`\`\`${path.extname(filePath).slice(1)}\n${content}\n\`\`\``,
    };
  }
}
```

We will make AI able to call 2 functions, namely `createFile` and `readFile`.

Next, we will create a schema so that AI can understand how to use our functions

```tsx
const createFileDeclaration: FunctionDeclaration = {
  name: "createFile",
  description: "Create a file with the given name and content.",
  parameters: {
    type: SchemaType.OBJECT,
    properties: {
      name: {
        type: SchemaType.STRING,
        description: "Name of the file to be created.",
      },
      content: {
        type: SchemaType.STRING,
        description: "Content to be written to the file.",
      },
    },
    required: ["name", "content"],
  },
};

const readFileDeclaration: FunctionDeclaration = {
  name: "readFile",
  description: "Read a file with the given name.",
  parameters: {
    type: SchemaType.OBJECT,
    properties: {
      name: {
        type: SchemaType.STRING,
        description: "Name of the file to be read.",
      },
    },
    required: ["name"],
  },
};

const functionDeclarations: FunctionDeclaration[] = [
  createFileDeclaration,
  readFileDeclaration,
];
```

We use OpenAPI as our schema, with this schema, AI will be able to understand how to use the functions we created.

Then we will call the model we will use, which is `gemini-1.5-flash`.

```ts
const API_KEY = "PUT_YOUR_API_KEY_HERE"
const genAI = new GoogleGenerativeAI(API_KEY);

const model = genAI.getGenerativeModel({
  model: "gemini-1.5-flash",
  tools: [
    {
      functionDeclarations
    }
  ]
});
```

Next, we'll create some *magic ✨*

```ts
const chat = model.startChat();
const prompt = "Create a file elephant.txt containing 'Elephant'";

const result = await chat.sendMessage(prompt)

// To make it easier, we'll use the first called function found
const calls = result.response.functionCalls();
const call = calls ? calls[0] : null;

if(call) {
  // @ts-ignore
  const funcResponse = await functions[call.name](call.args)
  
  const functionResult = await chat.sendMessage([
    {
      functionResponse: {
        name: call.name,
        response: funcResponse
      }
    }
  ])
  
  console.log(functionResult.response.text())
}
```

Done! We'll start executing the code and see the *magic* ✨.

```bash
bun run index.ts
```

Try to execute and see what happens

If everything goes smoothly, a file named `elephant.txt` containing `Elephant` should appear. Here's an example of the response:

```bash
Ok, I've created it. 😄
```

If there's an error, try changing the prompt. For some reason, sometimes with a different prompt but the same meaning, it works.

Now try changing the prompt to:

```ts
const prompt = "Read the elephant.txt file and explain its contents"
```

If there's no error, AI should read the file contents and explain them. Here's an example of the response:

```bash
The elephant.txt file contains the word "Elephant".
```

If there's an error, try changing the prompt.

I hope you found it helpful and informative. If you have any questions or need further clarification, please don't hesitate to ask.