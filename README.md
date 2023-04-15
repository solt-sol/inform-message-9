# Рабочий код в режиме ручного управления

Чтобы код работал корректно необходимо поместить файл с навзванием "pisk.mp3" в ту же папку, где и находится файл с кодом. 
Команды:
.start - подключения бота
.start_rec - запуск записи
.stop - отключение бота
.stop_rec - остановка записи


```
from enum import Enum
import discord
from discord.ext import commands
import asyncio
import speech_recognition as s_r
import whisper
import numpy as np
import torch
import requests
import ffmpeg
import io
from lxml import etree
from lxml import html
from gtts import gTTS
vc = ''
token=''
bot = commands.Bot(command_prefix = ["."], intents = discord.Intents.all(), help_command = None) # тут можно задать префикс
'''
intents = discord.Intents.default()
intents.message_content = True
bot = discord.Client(intents=intents)
'''
connections = {}
api_url = 'https://api.silero.ai/transcribe'


r = s_r.Recognizer()
def filt(text):
    ru_lats = "йцукенгшщзхфывапролджэячсмитьбюёъ"
    ru_lats += ru_lats.upper() + " "
    a = ''.join(filter(ru_lats.__contains__, text))
    print(a)
    return a

def text(text):
    google = "https://www.google.com/search?q="
    hearders = {
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 YaBrowser/23.1.5.708 Yowser/2.5 Safari/537.36'
    }
    params = text
    stork_list = params.split(" ")
    i = 0
    new_strok = ""
    for i in range(len(stork_list)):
        new_strok += stork_list[i] + "+"
        i += 1
    to_resp = google + new_strok[:-1]
    resp = requests.get(to_resp, headers=hearders)
    tree = html.fromstring(resp.content)
    b = []
    try:
        a = tree.xpath(".//div[contains(@id,'search')]/div/div[contains(@id,'rso')]/div[contains(@class,'hlcw0c')][1]/div[contains(@class,'MjjYud')]/div/div/div/ul/li/div/div/div/div[contains(@data-sncf,1)]/div/span//text()")
        print(1)
    except:
        print("hhtp1")
    if a == b:
        try:
            a = tree.xpath(".//div[contains(@id,'search')]/div/div[contains(@id,'rso')]/div[contains(@class,'hlcw0c')][1]/div[contains(@class,'MjjYud')]/div/div/div/ul/li/div/div/div/div[contains(@data-sncf,2)]/div/span//text()")
            print(2)
        except:
            print("http2")
    if a ==b:
        try:
            a = tree.xpath(".//div[contains(@id,'search')]/div/div[contains(@id,'rso')]/div[contains(@class,'MjjYud')][2]/div/div/div[contains(@data-content-feature,'')][2]/div/span//text()")
            print(3)
        except:
            print("http3")
    if a == b:
        try:
            a = tree.xpath(".//div[contains(@id,'search')]/div/div[contains(@id,'rso')]/div[contains(@class,'MjjYud')]/div/div/div[contains(@data-content-feature,'')][2]/div/span//text()")
            print(4)
        except:
            print("http4")
    i = 0
    strok = ''
    for i in range(len(a)):
        strok += a[i]
        i += 1
    text_to_ret = filt(strok)
    return text_to_ret


@bot.event
async def on_ready():
    print("Bot - Online.._") # эта функция воспроизводится один раз при запуске бота, после которой бот будет включен и будет реагировать на команды

@bot.command()
async def start_rec(ctx):
    global vc
    if ctx.message.author.voice: # проверка есть ли человек который вызвал эту команду в войс канале
        voice = ctx.author.voice
        channel = ctx.author.voice.channel
        voice1 = ctx.message.guild.voice_client
        if not voice1: # проверка нет ли бот в войс канале
            print('Бот не находится в войс канале')
            await ctx.send('Бот не находится в войс канале')
        else:
            if channel.id != voice1.channel.id: # проверкка на то, что бот и человек не в разынх каналах
                print('Бот и автор команды находятся в разных каналах')
                await ctx.send('Бот и автор команды находятся в разных каналах')
            else:
                vc.start_recording( # начало записис
                    #discord.sinks.MP3Sink(),  
                    discord.sinks.WaveSink(),
                    once_done,  
                    ctx.channel, 
                    ctx
                )
                print("Запись начата в канале '" + channel.name + "'")
                await vc.play(discord.FFmpegPCMAudio("pisk.mp3"))
    else:
        print("Автора команды нет в голосовом канале")
        await ctx.send("Автора команды нет в голосовом канале")

@bot.command()
async def start(ctx): # эта функция вызывается каждый когда мы пишем в текстовый чат команду ".start", т.к. у нас функция называется "start", а префикс "."
    global vc
    if ctx.message.author.voice: # проверка есть ли человек который вызвал эту команду в войс канале
        voice = ctx.author.voice
        channel = ctx.author.voice.channel
        voice1 = ctx.message.guild.voice_client
        if not voice1: # проверка нет ли бот в войс канале
            print("подключение...")
            vc = await voice.channel.connect()  # присоединение в войс канал
            connections.update({ctx.guild.id: vc}) 
            print("Старт в канале '" + channel.name + "'")
        else:
            if channel.id != voice1.channel.id: # проверкка на то, что бот и человек не в разынх каналах
                name_from = voice1.channel.name
                if ctx.guild.id in connections: # этот иф был в примере и оно так работает
                    del connections[ctx.guild.id] 
                    await vc.disconnect()
                    print("Стоп в канале '" + name_from + "'")
                    await asyncio.sleep(2)
                    vc = await voice.channel.connect()  
                    print("Перемещение из канала '" + name_from + "' в канал '" + channel.name + "'") # перемещение из одного войс канала в другой с сохранением записи
                    connections.update({ctx.guild.id: vc})  
                else:
                    print("Ошибка")
                    await ctx.send("Ошибка")
                #print("Начало записи в канале '" + channel.name + "'")
            else:
                print("Бот уже в этом канале - '" + channel.name + "'")
                await ctx.send("Бот уже в этом канале - '" + channel.name + "'")
    else:
        print("Автора команды нет в голосовом канале")
        await ctx.send("Автора команды нет в голосовом канале")

async def once_done(sink: discord.sinks, channel: discord.TextChannel, ctx, *args): # функция куда идет аудиофайл, а также его последующая обработка
    global vc
    recorded_users = [ #то, что можно удалить, но пока оно не мешает
        f"<@{user_id}>"
        for user_id, audio in sink.audio_data.items()
    ]
    #await sink.vc.disconnect() #отключение от дс канала
    files_out = [(audio.file, f"{user_id}.{sink.encoding}") for user_id, audio in sink.audio_data.items()]
    bytes_from_files = files_out[0][0]
    name_of_file = str(ctx.guild.id) + ".wav" 
    #files = [discord.File(audio.file, f"{user_id}.{sink.encoding}") for user_id, audio in sink.audio_data.items()]
    #print(files[0])
    #a = files[0]
    print(files_out, "\n\n\n\n\n\n\n\n\n\n\n")
    #d = files_out[0][0]
    #with open(d.getvalue(), "wb") as fil:
    #    fil.write(files_out[0].frame_data) #попытка сохранить файл (то что вам надо посмотреть)
    with open(name_of_file, "wb") as file_to_save:
        file_to_save.write(files_out[0][0].read())
    #file_audio = s_r.AudioFile(files_out[0][0])
    '''
    with file_audio as to_text: #
        data = r.record(to_text)                    #обязательный строчки 
        #textt = r.recognize_google(data, key = "AIzaSyCwdOLRMKE-quEGwui7UnbFXbwRcT5W7ok", language = 'ru', show_all = True)
        textt = r.recognize_whisper(data, model="base", language="russian") #результат текста из речи
        print(textt)
        #print(textt_2)
    '''
    model = whisper.load_model(name = "tiny")
    #e = np.array(files_out[0])
    #print(type(e))ъ
    #bytess = io.BytesIO()
    #bytess.seek(0)
    #arge = io.BytesIO(bytess)
    '''
    file_audio = np.frombuffer(files_out[0][0].getvalue(), dtype = np.int32)
    
    #file_audio.flags.writeable = True
    print(file_audio)
    
    result = model.transcribe(file_audio, fp16=False, language='Russian')
    '''
    audio = whisper.load_audio(name_of_file)
    #audio = np.frombuffer(files_out[0][0].getvalue(), dtype = np.int32)
    #audio = torch.tensor(audio)
    audio = whisper.pad_or_trim(audio)

    # make log-Mel spectrogram and move to the same device as the model
    mel = whisper.log_mel_spectrogram(audio).to(model.device)
    options = whisper.DecodingOptions(language="ru", fp16=False)
    result = whisper.decode(model, mel, options)

    # print the recognized text
    #print(result)
    print(result.text)
    #'''
    #textt = textt_0["text"]
    textt = text(result.text)
    tts = gTTS(textt, lang= "ru", tld= 'ru')
    nn_name_sound = "1" + str(ctx.guild.id) + ".mp3" 
    tts.save(nn_name_sound)
    await sink.vc.play(discord.FFmpegPCMAudio(nn_name_sound))
    #print(text_from, "\n")
    #print(text_from.text)
    files = []
    files.append(discord.File(bytes_from_files, name_of_file))
    #text = client.speech(a[0], {'Content-Type': 'audio/wav'})
    print(files)
    #await channel.send((textt + "\n\n" + textt_2), files=files)  
    await channel.send(textt + " //// " + result.text, files=files)  

@bot.command()
async def stop_rec(ctx):
    global vc
    channel = ctx.author.voice.channel
    voice = ctx.message.guild.voice_client
    if voice: # проверка есть ли бот в войс канале
        if ctx.author.voice: # есть ли автор команды в войс канале
            if channel.id != voice.channel.id: # проверка на то, в разных ли каналах бот и человек который вызвал команду
                print("Бот находится в другом канале - '" + voice.channel.name + "'")
                await ctx.send("Бот находится в другом канале - '" + voice.channel.name + "'")
            else:
                if ctx.guild.id in connections: # этот иф был в примере и оно так работает
                    ###vc = connections[ctx.guild.id]
                    vc.stop_recording() # стоп записи
                    print("Стоп записи в канале '" + channel.name + "'")
                else:
                    print("Ошибка")
                    await ctx.send("Ошибка")
        else:
            print("Автор сообщения не находится в голосовом канале")
            await ctx.send("Автор сообщения не находится в голосовом канале")
    else:
        print("Бот не играет")
        await ctx.send("Бот не играет")

@bot.command()
async def stop(ctx): # эта функция вызывается каждый когда мы пишем в текстовый чат команду ".stop", т.к. у нас функция называется "stop", а префикс "."
    global vc
    channel = ctx.author.voice.channel
    voice = ctx.message.guild.voice_client
    if voice: # проверка есть ли бот в войс канале
        if ctx.author.voice: # есть ли автор команды в войс канале
            if channel.id != voice.channel.id: # проверка на то, в разных ли каналах бот и человек который вызвал команду
                print("Бот находится в другом канале - '" + voice.channel.name + "'")
                await ctx.send("Бот находится в другом канале - '" + voice.channel.name + "'")
            else:
                if ctx.guild.id in connections: # этот иф был в примере и оно так работает
                    ###vc = connections[ctx.guild.id]
                    await vc.disconnect()
                    del connections[ctx.guild.id]  
                    print("Стоп в канале '" + channel.name + "'")
                else:
                    print("Ошибка")
                    await ctx.send("Ошибка")
        else:
            print("Автор сообщения не находится в голосовом канале")
            await ctx.send("Автор сообщения не находится в голосовом канале")
    else:
        print("Бот не играет")
        await ctx.send("Бот не играет")

@bot.command()
async def info(ctx): # тестовая функция на вывод всех данных который мы можем вообще получить о людях который есть в войс канале вместе с ботом
    i = 0
    if ctx.message.guild.voice_client != None:
        for i in range(len(ctx.message.guild.voice_client.channel.members)):
            print(str(ctx.message.guild.voice_client.channel.members[i]) + " - " + str(ctx.message.guild.voice_client.channel.members[i].voice + "\n================================"))
            i += 1
    else:
        print("Бот не играет")

bot.run(token) # старт бота
```
