# djs-message-commands

A utility package to help you construct and validate message commands for [discord.js](https://discord.js.org/#/).

## Background

Ever since discord.js v13, slash commands have been far superior for both the developers and users, and for good reason.

Using classic message commands, it becomes hard to:

-   Parse commands into reliable, consistent formats
-   Handle _way_ too many edge cases (e.g. spacing between each argument, missing arguments, etc.)
-   Validate argument types (dear god)
-   Restrict specific arguments to certain defined values (e.g. only allow certain roles, a specific number etc.)
-   Create/manage commands in a scalable way
-   Handle permission/role descrepancies
-   _and a lot more..._ you know what I'm talking about.

While these problems have been widely acknowledged by the community, they are still a pain to deal with, as other packages don't quite hit the mark in terms of ease of use, e.g. consistency with discord.js, robustness etc.

This package aims to provide a safe and easy way to manage, create, and validate message commands, with an architecture reminiscent of discord.js' slash command builders. It also includes additional utility components you may find useful.

> _Note: This package tries to be as unopinionated as possible, but the only caveat is that it follows [discord.js' guide on managing file structure,](https://discordjs.guide/creating-your-bot/command-handling.html#individual-command-files) which may not be what you use._

## Usage

Read the in-depth documentation [here]! (WIP)

How discord.js recommends structuring slash commands:

```js
// **/commands/slash/foo.js
module.exports = {
	builder: new SlashCommandBuilder().setName("foo").setDescription("bar"),

	execute: async interaction => {
		// some code here...
	},
};
```

This package follows a similar pattern:

```js
// **/commands/message/foo.js
module.exports = {
	builder: new MessageCommandBuilder().setName("foo").setDescription("bar"),

	execute: async (message, options) => {
		// some code here...
	},
};
```

```js
// index.js
const bot = new Client({
	intents: // intents...
});

const commands = new Collection();

// saving the commands defined in the 'commands' directory
for (const file of fs.readdirSync("./commands/message")) {
	const command = require(`./commands/message/${file}`);
	// use the builder's name as the key
	commands.set(command.builder.name, command);

	// set any aliases the command may have with the same builder
	for (const alias of command.builder.aliases) {
		commands.set(alias, command);
	}
}

bot.on("messageCreate", async message => {
	if (message.author.bot) return;

	const args = message.content.trim().split(/\s+/);
	// if the prefix doesn't match, ignore the message
	if (args[0].slice(0, PREFIX.length) !== PREFIX) return;

	const commandName = args[0].slice(PREFIX.length);
	const command = commands.get(commandName);

	if (!command) {
		// handle command not found
		return;
	};

	await message.channel.sendTyping();

	// get errors and parsed options
	const { errors, options } = command.builder.validate(message);

	if (errors.length > 0) {
		console.warn(errors);
		return;
	}

	try {
		await command.execute(message, options);
	}
	catch (err) {
		// handle execution error...
	}
})
```

### Handling Options

```js
// **/commands/message/foo.js
import { MessageCommandBuilder } from "djs-message-commands";

module.exports = {
	builder: new MessageCommandBuilder()
		.setName("foo")
		.setDescription("bar")
		.addStringOption(option =>
			option
				// you can name this however you want
				.setName("string-option")
				.setDescription("foo option description")
		)
		.addNumberOption(option =>
			option
				.setName("number-option")
				.setDescription("foo option description")
		)
		.addBooleanOption(option =>
			option
				.setName("boolean-option")
				.setDescription("foo option description")
		)
	),

	execute: async (message, options) => {
		/*
			use array destructuring to get each option
			variable name doesn't matter, but it's recommended to be consistent
			with the name that you defined in the builder
		*/
		const [stringOption, numberOption, booleanOption] = options;
	},
};
```

Using TypeScript:

```ts
// **/commands/message/foo.ts
import { MessageCommandBuilder } from "djs-message-commands";

module.exports = {
	builder: ...

	// 'options' parameter is of type: unknown[]
	execute: async (message, options) => {
		// assert types as you see fit
		const [stringOption, numberOption, booleanOption] = options as [string, number, boolean];
	},
};

```

## Features

-   Create robust and easily testable message commands
-   Uses a discord.js-esque builder system
-   Built-in parser to parse strings into numbers, booleans, mentionables, etc.

## Installation

yarn:

```
yarn add djs-message-commands
```

npm:

```
npm install djs-message-commands
```

## Contribution

If you have any suggestions, please open an issue or pull request on the [GitHub repository](https://github.com/Shockch4rge/djs-message-commands)!

## License

MIT
