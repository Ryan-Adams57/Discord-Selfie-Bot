# Discord-Selfie-Bot

React to selfies with flair using face detection

# DiscordSelfieBot

A Discord bot that reacts to selfies posted in channels with custom emojis and friendly comments using face detection.

## Features
- Detect selfies using OpenCV face detection
- Custom emoji responses
- Configurable channels

## Setup
1. Create a bot on [Discord Developer Portal](https://discord.com/developers)
2. Paste the token into `config.json`

## Run
```bash
python selfiebot.py

Requirements
Python 3.9+

discord.py

opencv-python


### 🧠 `selfiebot.py`
```python
import discord
import cv2
import aiohttp
import numpy as np
import io
import json

with open("config.json") as f:
    config = json.load(f)

intents = discord.Intents.default()
intents.message_content = True
client = discord.Client(intents=intents)

face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_default.xml')

async def is_selfie(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            if resp.status == 200:
                img_bytes = await resp.read()
                img_arr = np.asarray(bytearray(img_bytes), dtype=np.uint8)
                img = cv2.imdecode(img_arr, cv2.IMREAD_COLOR)
                gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
                faces = face_cascade.detectMultiScale(gray, 1.1, 4)
                return len(faces) > 0
    return False

@client.event
async def on_ready():
    print(f'Logged in as {client.user}')

@client.event
async def on_message(message):
    if message.attachments:
        for attachment in message.attachments:
            if attachment.filename.lower().endswith((".jpg", ".png", ".jpeg")):
                if await is_selfie(attachment.url):
                    await message.add_reaction("😊")
                    await message.channel.send(f"{message.author.mention} Nice selfie!")

client.run(config["token"])

