![Logo](./logo.png)

<img src=https://img.shields.io/badge/Python-3.8-blue.svg> <img src=https://img.shields.io/github/license/sync1211/Wavelink-disnake.svg>

A robust and powerful Lavalink wrapper for [Disnake](https://github.com/DisnakeDev/disnake)!

## Documentation
[Official Documentation](https://wavelink.readthedocs.io/en/latest/wavelink.html)

## Installation
The following commands are currently the valid ways of installing WaveLink.

**WaveLink requires Python 3.8+**

**Windows**

```sh
python -m pip install https://github.com/sync1211/Wavelink-Disnake/archive/refs/heads/master.zip
```

**Linux**

```sh
python3 -m pip install https://github.com/sync1211/Wavelink-Disnake/archive/refs/heads/master.zip
```

## Getting Started

A quick and easy bot example:

```py
    import disnake
    import wavelink
    from disnake.ext import commands


    class Bot(commands.Bot):

        def __init__(self):
            super(Bot, self).__init__(command_prefix=['audio ', 'wave ','aw '])

            self.add_cog(Music(self))

        async def on_ready(self):
            print(f'Logged in as {self.user.name} | {self.user.id}')


    class Music(commands.Cog):

        def __init__(self, bot):
            self.bot = bot

            if not hasattr(bot, 'wavelink'):
                self.bot.wavelink = wavelink.Client(bot=self.bot)

            self.bot.loop.create_task(self.start_nodes())

        async def start_nodes(self):
            await self.bot.wait_until_ready()

            # Initiate our nodes. For this example we will use one server.
            # Region should be a disnake guild.region e.g sydney or us_central (Though this is not technically required)
            await self.bot.wavelink.initiate_node(host='127.0.0.1',
                                                  port=2333,
                                                  rest_uri='http://127.0.0.1:2333',
                                                  password='youshallnotpass',
                                                  identifier='TEST',
                                                  region='us_central')

        @commands.command(name='connect')
        async def connect_(self, ctx, *, channel: disnake.VoiceChannel=None):
            if not channel:
                try:
                    channel = ctx.author.voice.channel
                except AttributeError:
                    raise disnake.DiscordException('No channel to join. Please either specify a valid channel or join one.')

            player = self.bot.wavelink.get_player(ctx.guild.id)
            await ctx.send(f'Connecting to **`{channel.name}`**')
            await player.connect(channel.id)

        @commands.command()
        async def play(self, ctx, *, query: str):
            tracks = await self.bot.wavelink.get_tracks(f'ytsearch:{query}')

            if not tracks:
                return await ctx.send('Could not find any songs with that query.')

            player = self.bot.wavelink.get_player(ctx.guild.id)
            if not player.is_connected:
                await ctx.invoke(self.connect_)

            await ctx.send(f'Added {str(tracks[0])} to the queue.')
            await player.play(tracks[0])


    bot = Bot()
    bot.run('TOKEN')
```
