---
title: "Github Actions와 Youtube API, Twitter API를 이용한 유튜브 영상 자동 트윗 (1)"
date: 2023-07-22T12:19:57+09:00
tags: [Youtube API, Github Actions, Python]
categories: [Tech]
---

유튜브 시작했는데 트위터에 홍보하고 싶었음.

기존에는 유튜브에 영상올리고 영상URL이랑 제목 복사해서 트위터에 올렸었음.

간단한 작업이지만 귀찮음.

그래서 Github Actions 사용해서 자동화함.

## 간단한 설명
Youtube API를 발급받고, 정보를 가져오는 python툴을 작성해서 테스트

## 1. Youtube API발급
GCP에 접속해서 새 프로젝트 생성
https://console.cloud.google.com/projectselector2/apis/dashboard?supportedpurview=project

생성 후에 `API및 서비스 사용설정` 에서 `Youtube Data API v3` 를 활성화

활성화 후 `사용사 인증 정보 만들기` 에서 `공개 데이터` 선택후 인증 정보 생성(영상 업로드등은 하지않고, 검색만 사용하기 때문에 공개 데이터를 설정)

## 2. Youtube API 를 사용해서 채널 검색
윈도우 cmd 등을 사용해서 google-api-python-client 설치

```
pip install google-api-python-client
```

### 유튜브 채널을 검색하는 코드

```Python
from googleapiclient.discovery import build
import json

DEVELOPER_KEY = "생성한 API KEY"
YOUTUBE_API_SERVICE_NAME = "youtube"
YOUTUBE_API_VERSION = "v3"

SEARCH_KEYWORD = "빠니보틀"

youtube = build(
    YOUTUBE_API_SERVICE_NAME,
    YOUTUBE_API_VERSION,
    developerKey=DEVELOPER_KEY
)

response = youtube.search().list(q=SEARCH_KEYWORD, part='id,snippet', maxResults=5).execute()

for item in response.get('items', []):
    if item['id']['kind'] != 'youtube#channel':
        continue
    print('#' * 20)
    print(json.dumps(item, indent=2, ensure_ascii=False))
    print('#' * 20)
```

### 실행 결과
검색 키워드는 빠니보틀로 설정하고 실행.

```
####################
{
  "kind": "youtube#searchResult",
  "etag": "w8nICUqz5m6JLzgLC8KDKS7t738",
  "id": {
    "kind": "youtube#channel",
    "channelId": "UCNhofiqfw5nl-NeDJkXtPvw"
  },
  "snippet": {
    "publishedAt": "2012-03-19T04:06:59Z",
    "channelId": "UCNhofiqfw5nl-NeDJkXtPvw",
    "title": "빠니보틀 Pani Bottle",
    "description": "여행합니다.",
    "thumbnails": {
      "default": {
        "url": "https://yt3.ggpht.com/lh7GiITaDKNdh6hi2GasqJrfi6AqDaj0qqR1UvVWGZPjltJmYzuftmA65KDy_Tltu6S8k82WkQ=s88-c-k-c0xffffffff-no-rj-mo"
      },
      "medium": {
        "url": "https://yt3.ggpht.com/lh7GiITaDKNdh6hi2GasqJrfi6AqDaj0qqR1UvVWGZPjltJmYzuftmA65KDy_Tltu6S8k82WkQ=s240-c-k-c0xffffffff-no-rj-mo"
      },
      "high": {
        "url": "https://yt3.ggpht.com/lh7GiITaDKNdh6hi2GasqJrfi6AqDaj0qqR1UvVWGZPjltJmYzuftmA65KDy_Tltu6S8k82WkQ=s800-c-k-c0xffffffff-no-rj-mo"
      }
    },
    "channelTitle": "빠니보틀 Pani Bottle",
    "liveBroadcastContent": "none",
    "publishTime": "2012-03-19T04:06:59Z"
  }
}
####################
```

### channelId와 channelTitle만 출력하는 코드

```Python
for item in response.get('items', []):
    if item['id']['kind'] != 'youtube#channel':
        continue
    channelId = item['id']['channelId']
    channelTitle = item['snippet']['title']
    print('#' * 20)
    print(f"Channel ID: {channelId}")
    print(f"Channel Title: {channelTitle}")
    print('#' * 20)
```

### 실행 결과

```
####################
Channel ID: UCNhofiqfw5nl-NeDJkXtPvw
Channel Title: 빠니보틀 Pani Bottle
####################
```

## 3. channelID를 사용해서 최신 영상의 정보 가져오기

### 빠니보틀 채널의 가장 최근 영상의 정보를 가져오는 코드

```Python
from googleapiclient.discovery import build
import json
from datetime import datetime

DEVELOPER_KEY = "생성한 API KEY"
YOUTUBE_API_SERVICE_NAME = "youtube"
YOUTUBE_API_VERSION = "v3"
CHANNEL_ID = "UCNhofiqfw5nl-NeDJkXtPvw" #빠니보틀의 유튜브 채널ID

youtube = build(
    YOUTUBE_API_SERVICE_NAME,
    YOUTUBE_API_VERSION,
    developerKey=DEVELOPER_KEY
)

# 채널id와 시간순으로 정렬해서 최신 영상의 정보를 가져옴
response = youtube.search().list(channelId=CHANNEL_ID, part='id,snippet', maxResults=1, order='date').execute()

for item in response.get('items', []):
    if item['id']['kind'] != 'youtube#video':
        continue
    print(json.dumps(item, indent=2, ensure_ascii=False))
```

### 실행 결과

```
{
  "kind": "youtube#searchResult",
  "etag": "olgnqjISECcEiQsEzuDhtvWM23A",
  "id": {
    "kind": "youtube#video",
    "videoId": "5TIgtb39KJ0"
  },
  "snippet": {
    "publishedAt": "2023-06-26T03:00:02Z",
    "channelId": "UCNhofiqfw5nl-NeDJkXtPvw",
    "title": "바다에서 사는 바자우 부족 마을에서의 멋진 하루 【인도네시아5】",
    "description": "인도네시아 끝! 떼리마 까씨 -------------------------------------- 카메라 Gopro11 Galaxy S21 Ultra Insta360 X3 ...",
    "thumbnails": {
      "default": {
        "url": "https://i.ytimg.com/vi/5TIgtb39KJ0/default.jpg",
        "width": 120,
        "height": 90
      },
      "medium": {
        "url": "https://i.ytimg.com/vi/5TIgtb39KJ0/mqdefault.jpg",
        "width": 320,
        "height": 180
      },
      "high": {
        "url": "https://i.ytimg.com/vi/5TIgtb39KJ0/hqdefault.jpg",
        "width": 480,
        "height": 360
      }
    },
    "channelTitle": "빠니보틀 Pani Bottle",
    "liveBroadcastContent": "none",
    "publishTime": "2023-06-26T03:00:02Z"
  }
}
```

### 영상의 정보를 활용해서 최근 1달 안에 업로드된 영상이 있으면 제목, url, 업로드 시간을 출력하도록 수정

```Python
from googleapiclient.discovery import build
from datetime import datetime
import json

DEVELOPER_KEY = "생성한 API KEY"
YOUTUBE_API_SERVICE_NAME = "youtube"
YOUTUBE_API_VERSION = "v3"
CHANNEL_ID = "UCNhofiqfw5nl-NeDJkXtPvw"

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

# 기간을 1달로 설정 뒤의 숫자를 수정해서 1시간 1주일 등으로 변경 가능
if now_ts-upload_ts<=24*3600*30:
    print("최근 영상: ",title,latest_url,upload_dt)
else:
    print("최근 영상 없음")
```

### 실행 결과
```
최근 영상:  바다에서 사는 바자우 부족 마을에서의 멋진 하루 【인도네시아5】 https://youtu.be/5TIgtb39KJ0 2023-06-26 03:00:02
```