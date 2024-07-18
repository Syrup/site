---
title: 'How to Create Discord Bot With Bun #1'
date: 2024-07-18T22:57:29+07:00
# weight: 1
tags: ["typescript", "bun"]
author: "Syrup"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: true
---

How to create a Discord bot using Bun:

1. Install the necessary dependencies:

```bash
bun add discord.js
```

2. Create a new file, let's say `bot.js`, and add the following code:

```javascript
const Discord = require('discord.js');
const client = new Discord.Client();

client.on('ready', () => {
  console.log(`Logged in as ${client.user.tag}!`);
});

client.on('message', msg => {
  if (msg.content === 'ping') {
    msg.reply('Pong!');
  }
});

client.login('YOUR_DISCORD_BOT_TOKEN');
```

3. Replace `'YOUR_DISCORD_BOT_TOKEN'` with your actual Discord bot token. You can obtain a token by creating a new bot on the Discord Developer Portal.

4. Save the file and run it using Node.js:

```bash
node bot.js
```

5. Your bot should now be online and ready to respond to the "ping" command with "Pong!".

Remember to customize the bot's behavior and add more functionality as needed.
