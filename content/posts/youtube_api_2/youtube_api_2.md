---
title: "Github Actions와 Youtube API, Twitter API를 이용한 유튜브 영상 자동 트윗 (2)"
date: 2023-07-23T16:08:20+09:00
tags: [Youtube API, Twitter API, Github Actions, Python]
categories: [Tech]
images: []
---

## 간단한 설명
유튜브에서 가져온 영상 정보를 트위터에 올리는 코드를 작성.

## 1. Twitter API발급
트위터 개발자 포털에 접속해서 API발급
https://developer.twitter.com/en/portal/dashboard

트위터 계정이 있으면 자동으로 `Projects & Apps` 에 앱이 등록되어있음.

앱을 클릭하고 `User authentication settings` 에서 사진 처럼 설정하고 저장.

`Redirect URL` 랑 `Website URL` 은 트위터url로 설정해도 문제없음.

![App permissions](../img_1.jpg)

![App info](../img_2.jpg)

설정 완료 후 앱의 `Keys and tokens` 에서 `API Key and Secret` , `Access Token and Secret` 를 `Regenerate` 해서 따로 저장해둠.

퍼미션도 `Read and Write` 인지 확인.

![Keys and tokens](../img_3.jpg)

## 2. Twitter API를 사용해서 트윗

윈도우 cmd 등을 사용해서 tweepy 설치

```
pip install tweepy
```

### 트윗하는 코드

```
import tweepy

consumer_key = "XXX"
consumer_secret = "XXX"
access_token = "XXX"
access_token_secret = "XXX"

client = tweepy.Client(
    consumer_key = consumer_key,
    consumer_secret = consumer_secret,
    access_token = access_token,
    access_token_secret = access_token_secret
)

text = f"""
test twit\n
test twit2\n
test twit3\n
"""

client.create_tweet(text = text)
```

### 실행 결과

![Twit Result](../img_4.jpg)

## 3. Youtube영상의 정보를 트윗

### 빠니보틀 채널에서 1달이내의 업로드된 가장 최신영상의 제목과 url을 트윗하는 코드

```
from googleapiclient.discovery import build
from datetime import datetime, timedelta
import tweepy

# 유튜브API 정보
DEVELOPER_KEY = "XXX"
YOUTUBE_API_SERVICE_NAME = "youtube"
YOUTUBE_API_VERSION = "v3"
CHANNEL_ID = "UCNhofiqfw5nl-NeDJkXtPvw"

# 트위터API 정보
CONSUMER_KEY = "XXX"
CONSUMER_SECRET = "XXX"
ACCESS_TOKEN = "XXX"
ACCESS_TOKEN_SECRET = "XXX"

youtube = build(
    YOUTUBE_API_SERVICE_NAME,
    YOUTUBE_API_VERSION,
    developerKey=DEVELOPER_KEY
)

response = youtube.search().list(channelId=CHANNEL_ID, part='id,snippet', maxResults=1, order='date').execute()

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
    consumer_key = CONSUMER_KEY,
    consumer_secret = CONSUMER_SECRET,
    access_token = ACCESS_TOKEN,
    access_token_secret = ACCESS_TOKEN_SECRET
)

def post_tweet(title,latest_url):
    text = f"""
    {title}\n
    {latest_url}\n
    """
    
    client.create_tweet(text = text)

if now_ts-upload_ts<=24*3600*30:
    print("New Video: ",title,latest_url)
    post_tweet(title,latest_url)
else:
    print("No New Video")
```

### 실행 결과

```
New Video:  바다에서 사는 바자우 부족 마을에서의 멋진 하루 【인도네시아5】 https://youtu.be/5TIgtb39KJ0
```

![Twit Result](../img_5.jpg)
