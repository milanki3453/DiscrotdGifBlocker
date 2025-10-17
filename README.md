# Discord Gif Blocker Bot
_Discord Gif Blocker Bot is a Python-powered moderation tool designed to automatically remove GIFs from your server’s chat. It can target specific users, helping to keep your community channels clean and focused._

_With admin-only controls, you decide who can activate or deactivate GIF removal at any time. The bot detects GIFs in uploads, embedded media, and message links (including popular platforms like Tenor and Giphy), ensuring comprehensive automatic filtering._



# ATTENTION — please read this message below carefully!

This bot was originally created by me for use in small servers, among friends and communities with a limited number of members. The design makes it easy to specify exactly which users will have their GIF messages automatically deleted.

If you want the bot to work with server roles (for example, to block GIFs for an entire role rather than individual users), please contact me and I’ll be happy to implement this feature!
If you have the necessary programming experience, you’re welcome to extend the bot for role support yourself.

Note:
Gif Blocker Bot is best suited for small to medium-sized Discord servers where you need precise, user-based GIF removal. For larger communities or advanced moderation needs, role-based filtering can be custom-built as needed.



# **Features:**
- Delete all GIFs, including:
  - Attachments
  - Embedded links (Tenor, Giphy, etc.)
  - Direct URLs in message content
- Enable/disable GIF filtering for any member
- Only allowed admins can use control commands


# **Setup & Launch:**
1. Install dependencies:
```pip install discord.py flask```

2. Add your bot token to environment variable DISCORD_TOKEN.

3. Add your Discord user ID(s) to the ADMIN_IDS list in main.py (example: ADMIN_IDS = [123456789012345678])

4. Run the bot:
   ```python filename.py```

# **How to use:**
- To start deleting GIFs from a user:
 ```!nogif @username```

_Only users from the ADMIN_IDS list can use this command!_



- To stop deleting GIFs:
  ```!okgif```
Only admins can use this command.

# **Admin-Only Commands**
If someone who is not listed in ADMIN_IDS tries to use !nogif or !okgif, the bot will respond:
> "You are not allowed to use this command!"

To edit the list of users who control the bot’s filter, just change the IDs in:
```ADMIN_IDS = [your_id, another_admin_id]```


# If you have any questions, suggestions, issues with usage, difficulties with setup, or encounter any bugs — feel free to reach out! I always try to reply and help everyone.

**I hope this bot will be useful to you. Good luck and enjoy using it!**

```import discord
from discord.ext import commands
import os

from flask import Flask
from threading import Thread

app = Flask('')

@app.route('/')
def home():
    return "I'm alive!"

def run_flask():
    app.run(host='0.0.0.0', port=8080)

def keep_alive():
    t = Thread(target=run_flask)
    t.start()

intents = discord.Intents.default()
intents.messages = True
intents.message_content = True
intents.guilds = True

bot = commands.Bot(command_prefix="!", intents=intents)

TARGET_USER_ID = None
NOGIF_MODE = False
ADMIN_IDS = [123456789012345678]  # <-- add your Discord ID here

def is_admin(ctx):
    return ctx.author.id in ADMIN_IDS

@bot.event
async def on_message(message):
    global TARGET_USER_ID, NOGIF_MODE
    if message.author == bot.user:
        return
    if NOGIF_MODE and TARGET_USER_ID and message.author.id == TARGET_USER_ID:
        for attachment in message.attachments:
            if attachment.url.endswith('.gif'):
                await message.delete()
                return
        for embed in message.embeds:
            if (embed.url and '.gif' in embed.url) or (embed.description and '.gif' in embed.description):
                await message.delete()
                return
        if '.gif' in message.content or 'tenor.com/view' in message.content or 'giphy.com/gifs' in message.content:
            await message.delete()
            return
    await bot.process_commands(message)

@bot.command(name='nogif')
async def nogif(ctx, user: discord.Member):
    if not is_admin(ctx):
        await ctx.send("You are not allowed to use this command!")
        return
    global TARGET_USER_ID, NOGIF_MODE
    TARGET_USER_ID = user.id
    NOGIF_MODE = True
    await ctx.send(f"GIF deletion enabled for: {user.mention}")

@bot.command(name='okgif')
async def okgif(ctx):
    if not is_admin(ctx):
        await ctx.send("You are not allowed to use this command!")
        return
    global NOGIF_MODE
    NOGIF_MODE = False
    await ctx.send("GIF deletion disabled.")

if __name__ == '__main__':
    keep_alive()
    bot.run(os.getenv("DISCORD_TOKEN"))
```

