---
title: "NodeJS and CLI: Command Line Interface Creation Made Easy"
date: 2024-02-06 03:02:00 +0400
image:
  path: /assets/img/posts/2024-02-06-nodejs-cli-implementation/cover.webp
categories: [Tools, CLI]
tags: [nodejs, command line, cli tool development, automation, scripting, nodejs cli, developer productivity, commander.js, cli project setup]
---

This tutorial was written for learning to create a simple command line interface (CLI) projects with NodeJS, build and test.

## What is a CLI and Why is It Useful in NodeJS?

### What is a Command Line Interface (CLI)?

A **C**ommand **L**ine **I**nterface (CLI) is a powerful tool that allows users to interact with computer programs by typing commands into a terminal or console. Unlike the graphical user interfaces (GUIs) we're used to, CLIs don't rely on graphical elements like buttons or icons. Instead, they provide a more direct, text-based method of interaction that can be incredibly efficient and flexible, especially for developers.

### Why Use a CLI in NodeJS?

Known for its efficiency and scalability in building network applications, NodeJS is a popular environment among developers. Integrating the CLI into NodeJS projects offers several benefits:

1. **Speed and Efficiency**: CLI tools in NodeJS can perform tasks much faster than GUI-based tools, as they eliminate the overhead of loading graphical elements.
2. **Automation**: They are excellent for automating repetitive tasks, making them a favorite for developers looking to streamline their workflow.
3. **Control and Precision**: CLI tools provide more control over the functionalities and are highly precise, allowing developers to execute complex tasks with simple commands.
4. **Resource-Friendly**: Being lightweight, they consume fewer system resources, which is particularly beneficial for server environments or low-resource devices.

## Prerequisites

You should install the following components before continuing:

1. [NodeJS](https://nodejs.org/en) (version 12 of higher) or install NVM ([MacOS](https://tecadmin.net/install-nvm-macos-with-homebrew/), [Windows](https://github.com/coreybutler/nvm-windows), etc.)
2. Install IDE, such as [Visual Studio Code](https://code.visualstudio.com/)

## Core Features: Overview

In my opinion, to better understand a new topic, we should work with a valid task with understandable conditions, but not something abstract like displaying the message "Hello World" by executing a specific command. For example, I will start by creating a user notes CLI called Noty.

Noty is a simple CLI tool for creating, listing, viewing, and deleting personal notes from the command line:

- **Creating Notes**: Implement the `create` command to add a new note. Notes can be stored as individual text files or as entries in a JSON file.
- **Listing Notes**: Implement the `list` command to display all note titles to the user.
- **Deleting Notes**: Implement the `delete` command to remove a note by title.
- **Viewing Notes**: Implement the `show` command to show a note by title.

```terminal
amadev@MacBook-Air ~ % noty create "Grocery List" --text "1. Milk 2. Tea 3. Bread"
Note 'Grocery List' created successfully.

amadev@MacBook-Air ~ % noty list
Your Notes:
  - Grocery List

amadev@MacBook-Air ~ % noty show "grocery list"
Title: grocery list
Created: 2/4/2024

1. Milk 2. Tea 3. Bread

amadev@MacBook-Air ~ % noty delete "grocery list"
Note 'Grocery List' deleted successfully.
```

In this article, I will only implement the `create` command as part of a simple example of creating a CLI solution.

## Set Up your Environment for NodeJS CLI Development

### Key Steps in Creating a Basic CLI Tool in NodeJS

In this section I'll describe all the steps to create a new NodeJS CLI project, install the necessary dependencies to write the code that will execute the command to show.

#### Step 1: Create a new NodeJS Project

The first thing to do is create a new directory `noty` for a project and use the command line to navigate to it.

```terminal
mkdir noty
```

```terminal
cd noty
```

Next, you should initialize a new NodeJS project using following command.

```terminal
npm init -y
```

This command will create a new file called `package.json` in the root directory of the project, which contains all metadata and dependencies.

#### Step 2: Install Dependencies

To implement our CLI, we will use the [commander](https://www.npmjs.com/package/commander) package, which helps parsing command line arguments and simplifies the process of building the project, and the [@iarna/toml](https://www.npmjs.com/package/@iarna/toml) package, which helps saving and reading information from the TOML file.

```terminal
npm i commander
```

```terminal
npm i @iarna/toml
```

#### Step 3: Create Main File and Add a new Command

In the project, create a new file named `index.js`with following command.

```terminal
> index.js
```

As an entry point for the CLI tool, this command creates an empty `index.js` file. We need to open it and add the note creation implementation. In the file, you can see how the instance of the commander is registered and how the declaratively defines all of the rules for the specific build command.

I used a TOML file for storing because it is relatively readable and the source file turns out to be the lightest in file size then YAML, JSON, XML.

```js
#!/usr/bin/env node

const { program } = require("commander");
const toml = require("@iarna/toml");
const config = require("./package.json");

const STORAGE_PATH = "./_s.toml";
const NOTES_KEY = "notes";

program.name(config.name)
	.version(config.version);

program.command("create")
	.alias("c")
	.description("Creates a new note.")
	.argument("<title>", "Title of the note.")
	.requiredOption("-t, --text <text>", "Text of the note")
	.action((title, { text }) => {
		if (!fs.existsSync(STORAGE_PATH))
			fs.writeFileSync(STORAGE_PATH, `[${NOTES_KEY}]`);

		const storage = toml.parse(fs.readFileSync(STORAGE_PATH));

		if (!NOTES_KEY in storage)
			storage[NOTES_KEY] = {};

		if (title in storage[NOTES_KEY]) {
			console.log(`Note '${title}' already exists.`);
			return;
		}

		storage[NOTES_KEY][title] = [(new Date()).toISOString(), text];
		fs.writeFileSync(STORAGE_PATH, toml.stringify(storage));

		console.log(`Note '${title}' created successfully.`);
	});

program.parse();
```
{: file='index.js'}

I want to look at this line `#!/usr/bin/env node`. It calls shebang. Shebang or hashbang is the first line of the file that tells the OS which interpreter to use, so when you try to run the `index.js` file, the OS will run it using NodeJS. I suggest you read ["NodeJS Shebang"](https://alexewerlof.medium.com/node-shebang-e1d4b02f731d) if you need more information.

> When the user invokes the CLI, `index.js` is executed by the NodeJS interpreter every time, so it is advisable to keep this information in mind when planning a CLI tool to make it easier to optimize your program.
{: .prompt-info}

At this step, we can call the CLI through NodeJS interpreter and check how it works. Run the following command:

```terminal
node index.js create "Grocery List" --text "1. Milk 2. Tea 3. Bread"
```

The message with the title of the created note should appear in the output:

```plaintext
Note 'Grocery List' created successfully.
```

#### Step 4: Set Up package.json

The executed command still doesn't look like a complete CLI, so we need to finish a preparatory process and add the `bin` field to the `package.json` file. Field comments have been written to the `_comment` field just above the required field.

```json
{
  "_comment": "package and command name.",
  "name": "noty",
  "version": "0.1.0",
  "main": "index.js",
  "_comment": "here we define path to executable file.",
  "bin": "index.js",
}
```
{: file='package.json'}

However, we can choose a specific CLI name that is different from the package name:

```json
{
  "_comment": "using @scoped or `noty-cli` package name.",
  "name": "@amadev/noty", 
  "version": "0.1.0",
  "main": "index.js",
  "bin": {
    "_comment": "noty - command name & index.js - path to executable file.",
    "noty": "index.js"
  },
}
```
{: file='package.json'}

### Test your NodeJS CLI Tool Locally

To test the CLI, we should install the package globally on the system using npm. Run following command:

```terminal
npm link
```

This command will install the package globally by creating a symlink from the global `node_modules` directory to the local project. After running this, we should be able to execute the command using the name we specified in the `bin` field.

>The `bin` field is primarily used for creating npm packages that provide command-line tools. If your project is not intended to be installed globally or used as a CLI tool, you may not need to use the `bin` field.
{: .prompt-tip}

Now we can call the CLI tool, just as if it were published.

```terminal
noty c "The Best C# Blog Posts" -t "- How to Build a URL Shortener With .NET"
```

> To unlink the package and remove the symlink to your local project, you will need to run the `npm unlink <package-name>` command.
{: .prompt-info}

You have successfully created a simple CLI application with NodeJS. To expand your skills, explore other features of the commander package. For example, you can create subcommands, handle errors, and add command parameters.

## Conclusion

We've learned how to make a simple CLI tool with NodeJS in this guide. We started with what a CLI is and why it's good for NodeJS projects. Then, we set up everything needed and made Noty, a tool for managing notes from the command line. This shows how CLI tools can make our work easier and faster.

But there's more to explore in making CLI tools. I'm excited to tell you about two more blog posts coming up:

- **Making CLI Tools with Oclif Framework in NodeJS**
- **Boosting CLI Tools with Rust**

Keep an eye out for these posts if you're interested in making more advanced CLI tools, whether with NodeJS or by adding Rust into the mix.

Thanks for reading about making CLI tools with me. I hope you're excited to learn more. See you in the next posts!
