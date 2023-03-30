# import tweepy
import discord
from discord.ext import commands

# Подключение к Twitter API
auth = tweepy.OAuthHandler("consumer_key", "consumer_secret")
auth.set_access_token("access_token", "access_token_secret")
api = tweepy.API(auth)

# Подключение к Discord API
bot = commands.Bot(command_prefix='!')
client = discord.Client()

@client.event
async def on_ready():
    print(f'{client.user} has connected to Discord!')

# Обработчик команды "!start"
@bot.command(name='start')
async def start_twitter_stream(ctx):
    # Отправляем сообщение в канал Discord
    await ctx.send('Starting Twitter stream...')

    # Создаем класс StreamListener
    class MyStreamListener(tweepy.StreamListener):
        def on_status(self, status):
            # Отправляем твит в канал Discord
            channel = client.get_channel(CHANNEL_ID)
            embed = discord.Embed(title=f'New tweet from {status.user.screen_name}', description=status.text, url=f'https://twitter.com/i/web/status/{status.id_str}', color=discord.Color.blue())
            embed.set_author(name=status.user.name, url=f'https://twitter.com/{status.user.screen_name}', icon_url=status.user.profile_image_url)
            embed.set_footer(text='Twitter', icon_url='https://abs.twimg.com/icons/apple-touch-icon-192x192.png')
            channel.send(embed=embed)

    # Создаем потоковое соединение с Twitter API
    my_stream_listener = MyStreamListener()
    my_stream = tweepy.Stream(auth=api.auth, listener=my_stream_listener)
    my_stream.filter(follow=["USER_ID"])

# Запускаем бота Discord
client.run('DISCORD_TOKEN')
