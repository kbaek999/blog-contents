---
title: "Github Actions와 Youtube API, Twitter API를 이용한 유튜브 영상 자동 트윗 (3)"
date: 2023-07-29T11:39:41+09:00
tags: [Youtube API, Twitter API, Github Actions, Python]
categories: [Tech]
images: []
---

## 간단한 설명
유튜브에서 가져온 영상 정보를 트위터에 올리는 코드를 작성.

작성한 코드를 github actions를 사용해서 자동으로 실행되도록 설정.

## Github Actions Secret설정
Github에서 레포지토리생성

생성 후 레포지토리에서 `Settings` > `Secrets and variables` > `Actions` 클릭 후 시크릿 변수생성

변수의 내용은 트위터와 유튜브의 API 정보를 입력

![Github Actions Secret](../actions_secret.JPG)

## 유튜브 API코드 수정

Github Actions Secret에서 설정한 변수를 사용하도록 변경

```python
from googleapiclient.discovery import build
from datetime import datetime, timedelta
import tweepy
import os

# 유튜브API 정보
YOUTUBE_DEVELOPER_KEY = os.environ['YOUTUBE_DEVELOPER_KEY']
YOUTUBE_API_SERVICE_NAME = "youtube"
YOUTUBE_API_VERSION = "v3"
YOUTUBE_CHANNEL_ID = os.environ['YOUTUBE_CHANNEL_ID']

# 트위터API 정보
TWITTER_CONSUMER_KEY = os.environ['TWITTER_CONSUMER_KEY']
TWITTER_CONSUMER_SECRET = os.environ['TWITTER_CONSUMER_SECRET']
TWITTER_ACCESS_TOKEN = os.environ['TWITTER_ACCESS_TOKEN']
TWITTER_ACCESS_TOKEN_SECRET = os.environ['TWITTER_ACCESS_TOKEN_SECRET']

youtube = build(
    YOUTUBE_API_SERVICE_NAME,
    YOUTUBE_API_VERSION,
    developerKey=YOUTUBE_DEVELOPER_KEY
)

response = youtube.search().list(channelId=YOUTUBE_CHANNEL_ID, part='id,snippet', maxResults=1, order='date').execute()

# 최근영상을 검색하고, vedeoId, 제목, 업로드 시간을 저장하고 영상의 url을 생성
for item in response.get('items', []):
    if item['id']['kind'] != 'youtube#video':
        continue
    videoId=item["id"]["videoId"]
    title=item["snippet"]["title"]
    publishTime=item["snippet"]["publishTime"]
    description=item["snippet"]["description"]
    latest_url=f"https://youtu.be/{videoId}"

# 코드 실행 시간
now = datetime.now()

# 코드 실행 시간을 unixtime으로 변환
now_ts = int(now.timestamp())

# 최근영상 업로드 시간
upload_time = publishTime

# 끝의 Z를 삭제
upload_time=upload_time[:-1]

# 업로드 시간을 문자열로 변환
upload_dt = datetime.fromisoformat(upload_time)

# 업로드 시간을 unixtime으로 변환
upload_ts = int(upload_dt.timestamp())

client = tweepy.Client(
    consumer_key = TWITTER_CONSUMER_KEY,
    consumer_secret = TWITTER_CONSUMER_SECRET,
    access_token = TWITTER_ACCESS_TOKEN,
    access_token_secret = TWITTER_ACCESS_TOKEN_SECRET
)

def post_tweet(title,latest_url):
    text = f"""
    {title}\n
    {latest_url}\n
    """
    
    client.create_tweet(text = text)

if now_ts-upload_ts<=24*3600*7:
    print("New Video: ",title,latest_url)
    post_tweet(title,latest_url)
else:
    print("No New Video")
```

## Github Actions설정

레포지토리의 `.github/workflows/xxx.yml` 파일을 생성 후 밑의 내용으로 작성

schedule 에 actions실행시간을 설정. 다만 schedule 의 경우 제시간에 실행되지 않는 경우가 많음

파이썬 코드실행시 Github Actions Secret의 변수를 사용할 수 있도록 env를 설정

```
name: Uploading YouTube video information to Twitter

on:
  schedule:
    - cron: '1 * * * *'

jobs:
  build:

    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        python-version: ["3.9"]

    steps:
    - uses: actions/checkout@v3
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v3
      with:
        python-version: ${{ matrix.python-version }}
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        python -m pip install google-api-python-client
        python -m pip install tweepy
        if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
    - name: Uploading YouTube video information to Twitter
      env: 
        YOUTUBE_DEVELOPER_KEY: ${{ secrets.YOUTUBE_DEVELOPER_KEY }}
        YOUTUBE_CHANNEL_ID: ${{ secrets.YOUTUBE_CHANNEL_ID_TEST }}
        TWITTER_CONSUMER_KEY: ${{ secrets.TWITTER_CONSUMER_KEY }}
        TWITTER_CONSUMER_SECRET: ${{ secrets.TWITTER_CONSUMER_SECRET }}
        TWITTER_ACCESS_TOKEN: ${{ secrets.TWITTER_ACCESS_TOKEN }}
        TWITTER_ACCESS_TOKEN_SECRET: ${{ secrets.TWITTER_ACCESS_TOKEN_SECRET }}
      run: python youtube_title_twit.py
```

실행 성공하면 이렇게 나옴

![Github Actions Secret](../github_actions.JPG)