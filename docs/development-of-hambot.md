---
layout: post
title:  "Development of Ham Bot"
---
## The Beginning (2021-2022)

Over the last couple of years (Starting Late 2021), I have experimented with Discord Bots. To learn more about how to code these bots I made a simple Bot I called "Ham Bot".
Ham Bot would turn into a true nightmare. Towards the end of my first attempt at making a Bot, I realized how painful it was and how ugly the code looked.

Here is a snippet of the original code showing some of the uglier more complicated sections.

```python
import json
import random
import urllib

import asyncpraw
import discord
from discord.ext import commands

intents = discord.Intents.default()
intents.members = True
client = commands.Bot(description='Ham Bot (For Friends)', command_prefix='//', intents=intents)
client.remove_command('help')
responses = {'hello': ('Hello! :wave:','Hi! :smiley:', 'Hello. How are you?', 'https://tenor.com/view/hello-there-private-from-penguins-of-madagascar-hi-wave-hey-there-gif-16043627'), 'bye': ('Bye! :wave:', 'Have Fun! :wave:', 'https://tenor.com/view/peace-out-im-out-bye-gif-14304356')}
waiting_users = {}
reddit = asyncpraw.Reddit(client_id='ID', client_secret='TOKEN', user_agent='Ham Bot')


@client.group(invoke_without_command=True)
async def help(ctx):
    em = discord.Embed(title = 'Help', description = "Type //help <command> for more info on a command.  <:ham:855696927798067247>")
    em.add_field(name = 'Moderation', value = 'say, role_message')
    em.add_field(name = 'Fun', value = 'ping, meme, rps, coinflip')
    em.add_field(name = 'Support', value = 'bug, suggest')
    await ctx.author.send(embed=em)


@client.command()
async def rps(ctx, *, args=''):
    player_choice = None
    if args == '':
        await ctx.send('You Need To Add \'Rock\', \'Paper\' or \'Scissors\'!')
    else:
        random_choice = random.choice([[':rock:', ':roll_of_paper:'], [':roll_of_paper:', ':scissors:'], [':scissors:', ':rock:']])
        args = args.strip(':')
        if args.lower()[0] == 'r':
            player_choice = ':rock:'
        elif args.lower()[0] == 'p':
            player_choice = ':roll_of_paper:'
        elif args.lower()[0] == 's':
            player_choice = ':scissors:'
        else:
            await ctx.send('Invalid Option!')
        if player_choice:
            await ctx.send(f'My {random_choice[0]} VS Your {player_choice}')
            if random_choice[0] == player_choice:
                await ctx.send('Draw!')
            elif random_choice[1] == player_choice:
                await ctx.send('You Win!')
            else:
                await ctx.send("I Win!")


@client.event
async def on_message(message):
    global waiting_users
    num = random.randint(1,1000)
    msg = message.content
    server_data = json.load(open('authorized_servers.json', 'r'))
    for data in server_data['store_messages']:
        try:
            if data == message.guild.id:
                print(str(message.author.name)+':', str(msg))
        except AttributeError:
            print(str(message.author.name)+':', str(msg))
            break
    msg = msg.lower()
    msg = msg.strip()
    if message.author == client.user:
        return
    if num == 1:
        await message.channel.send(f'You said: {message.content}')
    if msg.startswith('hello') or msg.startswith('hi'):
        if num < 100:
            await message.channel.send(random.choice(responses['hello']))
    elif msg.startswith('bye') and num < 100:
            await message.channel.send(random.choice(responses['bye']))
    if msg.__contains__('<@!853491184923181067>') or msg.__contains__('<@853491184923181067>'):
        msg = msg.replace('<@!853491184923181067>', '')
        msg = msg.replace('<@853491184923181067>', '')
        msg = msg.strip()
        if msg == '':
            await message.channel.send('How can I help? :thinking:')
        else:
            await message.channel.send('Would you like me to search the web for that?')
            msg = urllib.parse.quote(msg, safe='')
            search = ['searching', msg]
            waiting_users[message.author.name] = search
    elif msg.startswith('yes') and message.author.name in waiting_users:
        if 'searching' in waiting_users[message.author.name]:
            await message.channel.send('Searching Now :computer:')
            await message.channel.send('https://www.google.com/search?q='+waiting_users[message.author.name][-1])
            waiting_users.pop(str(message.author.name))
    elif message.author.name in waiting_users:
        waiting_users.pop(message.author.name)
    await client.process_commands(message)


client.run('TOKEN')
```

To be perfectly honest, my understanding of Python at that time was still new, and I did not know any tricks to make the code look prettier.

Moving on from this massacre of a program, I started working on two other Discord Bots around the same time.

- Minecraft Bot
- Music Bot

Simply put, Minecraft Bot was used to connect in game accounts to a players Discord account. This would allow users to automatically whitelist themselves,
and allow me to ban players for a given amount of time through Discord. This code, similar to that of Hambot would be an ugly mess of ~470 lines of code.

This was the code just for linking a Discord account to a Minecraft account.

```python
@slash.slash(guild_ids=[843291341185089566], name='link', description='Link Your Discord Account To Minecraft!', options=[create_option(name='username', description='Your Minecraft Username!', required=True, option_type=3)])
async def link(ctx: SlashContext, *, username=''):
    if not lock:
        passed = True
        if username.strip() == '':
            await ctx.send('Must Enter Username!')
        else:
            await ctx.send('Looking For Account!')
            with open('profiles.json', 'r') as fh:
                profiles = json.load(fh)
            if profiles.__contains__(str(ctx.author.id)):
                await ctx.send('You Already Have A Profile Linked!')
            else:
                try:
                    if UsernameToUUID(username.strip()).get_uuid() == '':
                        await ctx.send('No Account Found!')
                    else:
                        for profile in profiles:
                            if profiles[profile] == UsernameToUUID(username.strip()).get_uuid():
                                await ctx.send(f'Player Already Linked To {await client.fetch_user(int(profile))}!')
                                passed = False
                                break
                        if passed:
                            proc = subprocess.Popen(
                                'connect.bat', stdin=subprocess.PIPE, stdout=subprocess.PIPE)
                            out = proc.communicate(
                                f'whitelist add {username.strip()}'.encode())
                            if out.__contains__('Player is already whitelisted'):
                                await ctx.send('Profile Already Linked!')
                            else:
                                await ctx.send('Profile Linked, You Can Now Join The Server!')
                                uuid = UsernameToUUID(username.strip()).get_uuid()
                                profiles[str(ctx.author.id)] = uuid
                                json.dump(profiles, open('profiles.json', 'w'))
                except UnboundLocalError:
                    await ctx.send('No Account Found!')
    else:
        await ctx.send('Linking Locked!')
```

As for Music Bot, this bot was designed to replace the more popular Discord Bot [Rhythm](https://rythm.fm/) after YouTube shut them down.
It took a few weeks, but I managed to get a somewhat working Bot that downloaded the music to my computer and streamed it through a Voice Call.

Here is the entire source code.

```python
import asyncio
import os
from collections import defaultdict

import discord
from discord.ext import commands
from discord_slash import SlashCommand, SlashContext
from discord_slash.utils.manage_commands import create_choice, create_option
from pytube import YouTube, Search

intents = discord.Intents.all()
intents.members = True
waiting = []
stopping = []
client = commands.Bot(command_prefix='/', intents=intents)
client.remove_command('help')
slash = SlashCommand(client, sync_commands=True)
queue = defaultdict(dict)


@slash.slash(
    name='help',
    description='Displays the commands 🎵',
    options=[
        create_option(
            name='command',
            description='Displays additional help for commands 🎵',
            required=False,
            option_type=3,
            choices=[
                create_choice(
                    name='play',
                    value='play',
                ),
                create_choice(
                    name='stop',
                    value='stop',
                ),
                create_choice(
                    name='leave',
                    value='leave',
                ),
                create_choice(
                    name='queue',
                    value='queue',
                ),
                create_choice(
                    name='skip',
                    value='skip',
                )
            ]
        )
    ]
)
async def help(ctx: SlashContext, *, command=''):
    if command == 'play':
        em = discord.Embed(
            title='Play', description='Plays any song on YouTube. :musical_note:')
        em.add_field(name='**Syntax**', value='/play <YouTube link or Title>')
        await ctx.send(embed=em)
    elif command == 'stop':
        em = discord.Embed(
            title='Stop', description='Stops all songs including the ones in the queue. :musical_note:')
        em.add_field(name='**Syntax**', value='/stop')
        await ctx.send(embed=em)
    elif command == 'skip':
        em = discord.Embed(
            title='Skip', description='Skips the current playing song and plays next song in queue or stops playing if no queue. :musical_note:')
        em.add_field(name='**Syntax**', value='/skip')
        await ctx.send(embed=em)
    elif command == 'leave':
        em = discord.Embed(
            title='Leave', description='Leaves the call and stops all the songs including the ones in the queue. :musical_note:')
        em.add_field(name='**Syntax**', value='/leave')
        await ctx.send(embed=em)
    elif command == 'queue':
        em = discord.Embed(
            title='Queue', description='Displays the queue if it is available. :musical_note:')
        em.add_field(name='**Syntax**', value='/queue')
        await ctx.send(embed=em)
    else:
        em = discord.Embed(
            title='Help', description='Type /help <command> for more info on a command. :musical_note:')
        em.add_field(name='**Music**',
                     value='- play\n- stop\n- leave\n- queue\n- skip')
        await ctx.send(embed=em)


@slash.slash(name='play', description='Plays any song on YouTube. 🎵', options=[create_option(name='song', description='Song Name/URL!', required=True, option_type=3)])
async def play(ctx: SlashContext, *, song=''):
    error = False
    is_queue = True
    try:
        guild_id = str(ctx.guild.id)
    except (AttributeError, discord.ext.commands.errors.CommandInvokeError):
        await ctx.send('Must be in a server/guild to use this command!')
    else:
        try:
            channel = ctx.author.voice.channel
        except (AttributeError, discord.ext.commands.errors.CommandInvokeError):
            await ctx.send('Must be in a call to use this command!')
        else:
            try:    
                if channel and channel != ctx.voice_client.channel:
                    await ctx.send('Must be in same call to use this command!')
                    error = True
                else:
                    song_file = os.path.isfile(guild_id+'Using.mp3')
                    try:
                        if song_file:
                            os.remove(guild_id+'Using.mp3')
                    except PermissionError:
                        if waiting.__contains__(ctx.guild.id):
                            pass
                        else:
                            await ctx.send('Adding to queue! View queue with /queue')
                            is_queue = False
            except (AttributeError, discord.ext.commands.errors.CommandInvokeError):
                pass
            if not error:
                try:
                    vc = await channel.connect()
                except (discord.ext.commands.errors.CommandInvokeError, discord.errors.ClientException, asyncio.exceptions.TimeoutError):
                    vc = ctx.guild.voice_client
                if song == '':
                    await ctx.send('Please add a song link or name!')
                else:
                    if waiting.__contains__(ctx.guild.id):
                        await ctx.send('Please wait for query!')
                    else:
                        await ctx.send(f'Searching for "**{song}**"! :musical_note:')
                        search = Search(song)
                        try:
                            args = 'https://www.youtube.com/watch?v=' + \
                                search.results[0].video_id
                            yt = YouTube(args)
                            if yt.length <= 600:
                                too_long = False
                            else:
                                too_long = True
                                await ctx.send('Song too long, Try Again!')
                        except (IndexError, discord.ext.commands.errors.CommandInvokeError):
                            await ctx.send('Unable to find song, Try Again!')
                        else:
                            if not too_long:
                                if waiting.__contains__(ctx.guild.id):
                                    await ctx.send('Please wait for query!')
                                else:
                                    waiting.append(ctx.guild.id)
                                    async def download():
                                        audios = yt.streams.filter(only_audio=True)
                                        audios[0].download(f'.\downloads\{guild_id}')
                                    await client.loop.create_task(download())
                                    if is_queue:
                                        for file in os.listdir(f'.\downloads\{guild_id}'):
                                            os.system(
                                                f'move ".\downloads\{guild_id}\{file}" ./')
                                            for music_file in os.listdir('./'):
                                                if not music_file.endswith('Using.mp3') and not music_file.endswith('Queue.mp3'):
                                                    if music_file.endswith('.mp4') or music_file.endswith('.webm') or music_file.endswith('.ogg') or music_file.endswith('.mp3'):
                                                        os.rename(
                                                            music_file, guild_id+'Using.mp3')
                                                if music_file.endswith('.webp') or music_file.endswith('.png') or music_file.endswith('.jpg'):
                                                    os.system(
                                                        f'del /f "{music_file}"')
                                        try:
                                            vc.play(discord.FFmpegPCMAudio(
                                                guild_id+'Using.mp3'))
                                            vc.source = discord.PCMVolumeTransformer(
                                                vc.source)
                                            vc.source.volume = 0.1
                                        except (discord.ext.commands.errors.CommandInvokeError, discord.errors.ClientException):
                                            await ctx.send('Unable to join/play in call!')
                                            os.system(
                                                f'del /f {guild_id}Using.mp3')
                                            try:
                                                waiting.remove(ctx.guild.id)
                                            except ValueError:
                                                pass
                                        else:
                                            try:
                                                em = discord.Embed(
                                                    title='Now Playing:')
                                                em.add_field(
                                                    name=yt.title, value=args)
                                                await ctx.send(embed=em)
                                                waiting.remove(ctx.guild.id)
                                            except ValueError:
                                                pass
                                    else:
                                        for file in os.listdir(f'.\downloads\{guild_id}'):
                                            os.system(
                                                f'move ".\downloads\{guild_id}\{file}" ./')
                                            for music_file in os.listdir('./'):
                                                if not music_file.endswith('Using.mp3') and not music_file.endswith('Queue.mp3'):
                                                    if music_file.endswith('.mp4') or music_file.endswith('.webm') or music_file.endswith('.ogg') or music_file.endswith('.mp3'):
                                                        os.rename(
                                                            music_file, guild_id+f'[{len(queue[ctx.guild.id])+1}]Queue.mp3')
                                                if music_file.endswith('.webp') or music_file.endswith('.png') or music_file.endswith('.jpg'):
                                                    os.system(
                                                        f'del /f "{music_file}"')
                                        try:
                                            queue[ctx.guild.id][len(queue[ctx.guild.id])+1] = {
                                                'title': yt.title, 'position': len(queue[ctx.guild.id])+1, 'url': args, 'ctx': ctx, 'channel': channel}
                                            waiting.remove(ctx.guild.id)
                                            await ctx.send(f'**{yt.title}** Added to queue!')
                                        except AttributeError:
                                            await ctx.send('User left Channel!')


@slash.slash(name='stop', description='Stops all songs including the ones in the queue. 🎵')
async def stop(ctx: SlashContext):
    try:
        try:
            _ = ctx.guild.id
        except (AttributeError, discord.ext.commands.errors.CommandInvokeError):
            await ctx.send('Must be in a server/guild to use this command!')
        else:
            voice = discord.utils.get(client.voice_clients, guild=ctx.guild)
            if ctx.author.voice.channel is None:
                await ctx.send('Must be in a call to use this command!')
            elif ctx.author.voice.channel and ctx.author.voice.channel == ctx.voice_client.channel:
                if voice.is_connected():
                    stopping.append(str(ctx.guild.id))
                    voice.stop()
                    await asyncio.sleep(0.2)
                    os.system(f'del /f {ctx.guild.id}Using.mp3')
                    for i in range(len(queue[int(ctx.guild.id)])):
                        os.system(f'del /f {ctx.guild.id}[{i+1}]Queue.mp3')
                    queue.pop(int(ctx.guild.id))
                    try:
                        waiting.remove(ctx.guild.id)
                    except ValueError:
                        pass
                    stopping.remove(str(ctx.guild.id))
                    await ctx.send('I have stopped playing music!')
                else:
                    await ctx.send('I must be in a call to use this command!')
    except (discord.ext.commands.errors.CommandInvokeError, AttributeError):
        await ctx.send('Must be in the same call to use this command!')


@slash.slash(name='skip', description='Skips the current playing song and plays next song in queue or stops playing if no queue. 🎵')
async def skip(ctx: SlashContext):
    try:
        try:
            _ = ctx.guild.id
        except (AttributeError, discord.ext.commands.errors.CommandInvokeError):
            await ctx.send('Must be in a server/guild to use this command!')
        else:
            voice = discord.utils.get(client.voice_clients, guild=ctx.guild)
            if ctx.author.voice.channel is None:
                await ctx.send('Must be in a call to use this command!')
            elif ctx.author.voice.channel and ctx.author.voice.channel == ctx.voice_client.channel:
                if voice.is_connected():
                    voice.stop()
                    await asyncio.sleep(0.2)
                    await ctx.send('Song skipped!')
                else:
                    await ctx.send('I must be in a call to use this command!')
    except (discord.ext.commands.errors.CommandInvokeError, AttributeError):
        await ctx.send('Must be in the same call to use this command!')


@slash.slash(name='leave', description='Leaves the call and stops all the songs including the ones in the queue. 🎵')
async def leave(ctx: SlashContext):
    try:
        try:
            _ = ctx.guild.id
        except (AttributeError, discord.ext.commands.errors.CommandInvokeError):
            await ctx.send('Must be in a server/guild to use this command!')
        else:
            voice = discord.utils.get(client.voice_clients, guild=ctx.guild)
            if ctx.author.voice.channel is None:
                await ctx.send('Must be in a call to use this command!')
            elif ctx.author.voice.channel and ctx.author.voice.channel == ctx.voice_client.channel:
                if voice.is_connected():
                    stopping.append(str(ctx.guild.id))
                    voice.stop()
                    await asyncio.sleep(0.2)
                    os.system(f'del /f {ctx.guild.id}Using.mp3')
                    for i in range(len(queue[int(ctx.guild.id)])):
                        os.system(f'del /f {ctx.guild.id}[{i+1}]Queue.mp3')
                    queue.pop(int(ctx.guild.id))
                    try:
                        waiting.remove(ctx.guild.id)
                    except ValueError:
                        pass
                    stopping.remove(str(ctx.guild.id))
                    await voice.disconnect()
                    await ctx.send('I have left the call and stopped playing music!')
                else:
                    await ctx.send('I must be in a call to use this command!')
    except (discord.ext.commands.errors.CommandInvokeError, AttributeError):
        await ctx.send('Must be in the same call to use this command!')


@slash.slash(name='queue', description='Displays the queue if it is available. 🎵')
async def _queue(ctx: SlashContext):
    song = None
    em = discord.Embed(
        title='Queue', description='List of queued songs. :musical_note:')
    for song in queue[ctx.guild.id]:
        em.add_field(name=f'{queue[ctx.guild.id][song]["position"]}. {queue[ctx.guild.id][song]["title"]}',
                     value=queue[ctx.guild.id][song]['url'])
    if song:
        await ctx.send(embed=em)
    else:
        await ctx.send('No queue found!')


@client.event
async def on_voice_state_update(member, before, after):
    try:
        voice = discord.utils.get(client.voice_clients, guild=member.guild)
        if member == client.user:
            try:
                _ = after.channel.id
                if before.channel.id != after.channel.id:
                    voice.stop()
                    await asyncio.sleep(0.2)
                    os.system(f'del /f {member.guild.id}Using.mp3')
                    try:
                        waiting.remove(member.guild.id)
                    except ValueError:
                        pass
            except (AttributeError):
                try:
                    _ = before.channel.id
                except (AttributeError):
                    pass
                else:
                    voice.stop()
                    await asyncio.sleep(0.2)
                    os.system(f'del /f {member.guild.id}Using.mp3')
                    try:
                        waiting.remove(member.guild.id)
                    except ValueError:
                        pass
    except (AttributeError):
        os.system(f'del /f {member.guild.id}Using.mp3')


async def play_queue(ctx, guild_id):
    channel = ctx['channel']
    try:
        vc = await channel.connect()
    except (discord.ext.commands.errors.CommandInvokeError, discord.errors.ClientException, asyncio.exceptions.TimeoutError):
        vc = ctx['ctx'].guild.voice_client
    try:
        vc.play(discord.FFmpegPCMAudio(str(guild_id)+'Using.mp3'))
        vc.source = discord.PCMVolumeTransformer(vc.source)
        vc.source.volume = 0.1
    except (discord.ext.commands.errors.CommandInvokeError, discord.errors.ClientException):
        await asyncio.sleep(1)
        vc = ctx['ctx'].guild.voice_client
        try:
            vc.play(discord.FFmpegPCMAudio(str(guild_id)+'Using.mp3'))
            vc.source = discord.PCMVolumeTransformer(vc.source)
            vc.source.volume = 0.1
        except (discord.ext.commands.errors.CommandInvokeError, discord.errors.ClientException):
            await ctx['ctx'].send('Unable to join/play in call, skipping song!')
    else:
        em = discord.Embed(title='Now Playing:')
        em.add_field(name=ctx['title'], value=ctx['url'])
        await ctx['ctx'].send(embed=em)
        await asyncio.sleep(0.2)


async def get_queue():
    await asyncio.sleep(5)
    while True:
        await asyncio.sleep(0.1)
        for file in os.listdir('./'):
            if file.endswith('Queue.mp3'):
                guild_id = file.split('[')[0]
                position = int(file.split('[')[1].split(']')[0])
                song = os.path.isfile(guild_id+'Using.mp3')
                try:
                    if song and not stopping.__contains__(guild_id):
                        os.remove(guild_id+'Using.mp3')
                        if not stopping.__contains__(guild_id) and position == 1:
                            os.rename(file, guild_id+'Using.mp3')
                            await play_queue(queue[int(guild_id)][1], int(guild_id))
                            await asyncio.sleep(1)
                            queue[int(guild_id)].pop(1)
                            num = 1
                            while True:
                                await asyncio.sleep(0.1)
                                num += 1
                                try:
                                    if num-1 <= len(queue[int(guild_id)]):
                                        os.rename(
                                            f'{guild_id}[{num}]Queue.mp3', f'{guild_id}[{num-1}]Queue.mp3')
                                        queue[int(guild_id)
                                              ][num]['position'] = num-1
                                        queue[int(guild_id)][num -
                                                             1] = queue[int(guild_id)].pop(num)
                                    else:
                                        break
                                except WindowsError:
                                    pass
                except PermissionError:
                    pass


@client.event
async def on_ready():
    try:
        for file in os.listdir('./'):
            if file.endswith('.mp3') or file.endswith('.webp') or file.endswith('.mp4') or file.endswith('.png') or file.endswith('.jpg') or file.endswith('.webm'):
                os.remove(file)
        os.system('rmdir /s /q downloads')
    except (FileNotFoundError, PermissionError):
        pass
    await client.change_presence(activity=discord.Game(name="/help to view commands! 🎵"))
    print('Connected')


client.loop.create_task(get_queue())
client.run('TOKEN')

```

After a few years, both Bots would see an unfortunate demise after API changes and me no longer having the effort to maintain them. However,
since Ham Bot did not rely on many API calls (Besides the Reddit API), it still runs to this day while I develop the better version.

## The Better Version (2023-Present)

After a couple years of more advanced experience I was ready to go back to working on Discord Bots, where I decided to improve all of the Bots.
Although it was at this point I realized a lot of the stuff I had learned before no longer was relevant due to a Syntax and API rewrite in newer versions of discord.py (Which also broke the older bots completely).
First things first, I started planning how I wanted these new Bots to work. I decided on a website that would allow direct access to Bot settings for individual Servers would be best.
The website I went with was designed using [Streamlit](https://streamlit.io/), which allowed me to use Python to build the website and interact with JSON files. For the Bot side of things,
the Bot when asked to run a command would check through the JSON file for that Server and run the command based on that data.

Referring back to the first code of Ham Bot (Now called Ham Bot Original), this is the new version of the Rock Paper Scissors Command.

```python
# Required Imports
import discord
from discord import app_commands
from discord.ext import commands
import os
from custom.check_command_allowed import check_command_allowed
from custom import errors

# Command Imports
import random
import asyncio

# Command Name
command = os.path.basename(__file__)[:-3]

# Cog for the command to be loaded into the bot
class Rps(commands.Cog):
    def __init__(self, bot: commands.Bot):
        self.bot = bot

    @app_commands.command(name=command, description='Play Paper, Scissors, Rock ✂️')
    @app_commands.guild_only()
    @app_commands.describe(choice='Choose what to play')
    @app_commands.choices(choice=[
        app_commands.Choice(name='Paper', value=':page_facing_up:'),
        app_commands.Choice(name='Scissors', value=':scissors:'),
        app_commands.Choice(name='Rock', value=':rock:')
    ])
    async def rps(self, interaction: discord.Interaction, choice: app_commands.Choice[str]):
        try:
            if (x := check_command_allowed(interaction, command)) is True:
                random_choice = random.choice([[':rock:', ':page_facing_up:'], [':page_facing_up:', ':scissors:'], [':scissors:', ':rock:']])
                await self.bot.get_channel(interaction.channel_id).typing()
                await interaction.response.send_message('Paper!')
                await asyncio.sleep(0.125)
                await interaction.edit_original_response(content='Scissors!')
                await asyncio.sleep(0.125)
                await interaction.edit_original_response(content='Rock!')
                await asyncio.sleep(0.125)
                await interaction.edit_original_response(content='Shoot!')
                await asyncio.sleep(0.125)
                if random_choice[0] == choice.value:
                    await interaction.edit_original_response(content=f'**My {random_choice[0]} VS Your {choice.value}**\nDraw!')
                elif random_choice[1] == choice.value:
                    await interaction.edit_original_response(content=f'**My {random_choice[0]} VS Your {choice.value}**\nYou Win! 👑')
                else:
                    await interaction.edit_original_response(content=f'**My {random_choice[0]} VS Your {choice.value}**\nI Win!')
            else:
                await errors.command(interaction, x, command)
        except KeyError:
            await errors.command_not_found(interaction, command)

async def setup(bot):
    await bot.add_cog(Rps(bot))
```

Notice something special about this version? It uses a class object called RPS. In fact every command uses a structure similar to this now.
This is because loading modules from different scripts helps keep the Main Script clean.

Which is clearly shown here, as this is the Main Script.

```python
# Discord Imports
import discord
from discord.ext import commands

# Main Imports
import asyncio
import os

# Custom Imports
from custom import config

# Create the client class to load the bot
class Client(commands.Bot):
    def __init__(self):
        intents = discord.Intents.all()
        intents.message_content = True
        intents.members = True
        super().__init__(command_prefix=None, intents=intents)

    async def on_command_error(self, ctx, error):
        await ctx.reply(error, ephemeral=True)


# Initialize the Client
client = Client()

# Any global variables
guilds_not_found = {}

# Clear the screen
if os.name == 'posix':
    os.system('clear')
else:
    os.system('cls')


async def load():
    for file in os.listdir('./addons/commands'):
        if file.endswith('.py'):
            await client.load_extension(f'addons.commands.{file[:-3]}')
    for file in os.listdir('./addons/events'):
        if file.endswith('.py'):
            await client.load_extension(f'addons.events.{file[:-3]}')


async def main():
    await load()
    await client.start(config.TOKEN)


asyncio.run(main())
```

The only issue I faced with this new method of creating Discord Bots was maintaining the large amount of files. Just for Ham Bot currently there are currently 49 files being used.

This includes.

- Bot Commands
- Bot Events
- Other Bot stuff
  - Main Script
  - Template Scripts/Generator Scripts
- Website stuff
  - Authentication stuff
  - Different pages
  - Custom Addons
- Website Bot Handling

My most recent addition to the new Ham Bot was adding the Music feature back. This time, I made the whole feature faster and more reliable, with only a few noted issues.
The biggest change introduced in this version was how Music was played, instead of downloading to my computer, it would now directly stream the music from YouTube.

This is the code for the play command
```python
# Required Imports
import discord
from discord import app_commands
from discord.ext import commands
import os
from custom.check_command_allowed import check_command_allowed
from custom import errors

# Command Imports
import yt_dlp
import aiohttp
import json
import asyncio
import logging

# Command Name
command = os.path.basename(__file__)[:-3]

# Global Variables
FFMPEG_OPTS = {'before_options': '-reconnect 1 -reconnect_streamed 1 -reconnect_delay_max 5', 'options': '-vn'}
yt_dlp_logger = logging.getLogger("ytdl-ignore")
yt_dlp_logger.disabled = True
session = aiohttp.ClientSession()

# Cog for the command to be loaded into the bot
class Play(commands.Cog):
    def __init__(self, bot: commands.Bot):
        self.bot = bot

    @app_commands.command(name=command, description='Play any song on YouTube. 🎵')
    @app_commands.guild_only()
    @app_commands.describe(song='Song Name/URL!')
    async def play(self, interaction: discord.Interaction, song: str):
        try:
            if (x := check_command_allowed(interaction, command)) is True:
                await interaction.response.defer()
                user_vc = interaction.user.voice
                if not user_vc:
                    await interaction.followup.send('You must be in a call to use this command!')
                    return
                bot_vc = interaction.guild.voice_client
                if bot_vc and (bot_vc.channel.id != user_vc.channel.id):
                    await interaction.followup.send('You must be in same call to use this command!')
                    return
                else:
                    with open(f'./guild_settings/{interaction.guild.id}/settings.json', 'r') as fh:
                        data = json.load(fh)
                        data.setdefault('song_manager', {'queue': 'inactive','songs': [], 'status': 'unlocked'})
                    if bot_vc:
                        vc = bot_vc
                    else:
                        data['song_manager']['status'] = 'unlocked'
                        data['song_manager']['queue'] = 'inactive'
                        data['song_manager']['songs'] = []
                        vc = await user_vc.channel.connect(self_deaf=True)
                if data['song_manager']['status'] == 'unlocked':
                    data['song_manager']['status'] = 'locked'
                    with open(f'./guild_settings/{interaction.guild.id}/settings.json', 'w') as fh:
                        json.dump(data, fh, indent=4, sort_keys=True)
                    await interaction.followup.send(f'Searching for "**{song}**"! 🎵')
                    with yt_dlp.YoutubeDL({'format': 'bestaudio', 'noplaylist':'True', 'quiet': 'True', 'logger': yt_dlp_logger}) as ydl:
                        try:
                            async with session.get(song):
                                pass
                            valid_links = ('youtube.com/watch?v=', 'youtu.be', 'm.youtube.com/watch?v=')
                            song_stripped = song.lstrip('https://').lstrip('www.').rstrip('/')
                            if not song_stripped.startswith(valid_links) or any(x == song_stripped for x in valid_links):
                                await interaction.followup.send('Invalid Link!')
                                data['song_manager']['status'] = 'unlocked'
                                with open(f'./guild_settings/{interaction.guild.id}/settings.json', 'w') as fh:
                                    json.dump(data, fh, indent=4, sort_keys=True)
                                return
                            info = ydl.extract_info(song, download=False)
                        except Exception:
                            try:
                                info = ydl.extract_info(f'ytsearch:{song}', download=False)['entries'][0]
                            except IndexError:
                                await interaction.followup.send('Unable to locate a video matching that search!')
                                data['song_manager']['status'] = 'unlocked'
                                with open(f'./guild_settings/{interaction.guild.id}/settings.json', 'w') as fh:
                                    json.dump(data, fh, indent=4, sort_keys=True)
                                return
                    data['song_manager']['songs'].append({'title': info['title'], 'thumbnail': info['thumbnail'], 'url': info['url'], 'original_url': info['original_url']})
                    data['song_manager']['status'] = 'unlocked'
                    with open(f'./guild_settings/{interaction.guild.id}/settings.json', 'w') as fh:
                        json.dump(data, fh, indent=4, sort_keys=True)
                    if vc.is_playing():
                        await interaction.followup.send(f'"**{info["title"]}**" Added to queue! 🎵')
                        await play_queue(interaction)
                    else:
                        source = data['song_manager']['songs'][0]['url']
                        vc.play(discord.FFmpegPCMAudio(source, **FFMPEG_OPTS))
                        em = discord.Embed(title='Now Playing:')
                        em.set_image(url=data['song_manager']['songs'][0]['thumbnail'])
                        em.add_field(name=data['song_manager']['songs'][0]['title'], value=data['song_manager']['songs'][0]['original_url'])
                        await interaction.channel.send(embed=em)
                        data['song_manager']['songs'].pop(0)
                        with open(f'./guild_settings/{interaction.guild.id}/settings.json', 'w') as fh:
                            json.dump(data, fh, indent=4, sort_keys=True)
                else:
                    await interaction.followup.send('You can only search one song at a time!')
            else:
                await errors.command(interaction, x, command)
        except KeyError:
            await errors.command_not_found(interaction, command)


async def setup(bot):
    await bot.add_cog(Play(bot))


async def play_queue(interaction):
    vc = interaction.guild.voice_client
    with open(f'./guild_settings/{interaction.guild.id}/settings.json', 'r') as fh:
        data = json.load(fh)
    if data['song_manager']['queue'] == 'active':
        return
    else:
        data['song_manager']['queue'] = 'active'
        with open(f'./guild_settings/{interaction.guild.id}/settings.json', 'w') as fh:
            json.dump(data, fh, indent=4, sort_keys=True)
    while True:
        await asyncio.sleep(1)
        with open(f'./guild_settings/{interaction.guild.id}/settings.json', 'r') as fh:
            data = json.load(fh)
        if not vc.is_playing() and interaction.guild.voice_client:
            source = data['song_manager']['songs'][0]['url']
            vc.play(discord.FFmpegPCMAudio(source, **FFMPEG_OPTS))
            em = discord.Embed(title='Now Playing:')
            em.set_image(url=data['song_manager']['songs'][0]['thumbnail'])
            em.add_field(name=data['song_manager']['songs'][0]['title'], value=data['song_manager']['songs'][0]['original_url'])
            await interaction.channel.send(embed=em)
            data['song_manager']['songs'].pop(0)
            with open(f'./guild_settings/{interaction.guild.id}/settings.json', 'w') as fh:
                json.dump(data, fh, indent=4, sort_keys=True)
        if len(data['song_manager']['songs']) == 0:
            data['song_manager']['queue'] = 'inactive'
            with open(f'./guild_settings/{interaction.guild.id}/settings.json', 'w') as fh:
                json.dump(data, fh, indent=4, sort_keys=True)
```

As of now, development of Ham Bot has stopped while I focus on my University Work. But I am proud of what I have done over the last few years improving and learning more about Discord Bots.
Ham Bot is not Available to the Public currently due to the website being unavailable (Which is necessary for Ham Bot to work). I am unsure if this will ever be available as Ham Bot is just a learning tool for me.
