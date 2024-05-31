# YouTube Channel Video Fetcher

This repository contains a Jupyter Notebook that uses the YouTube Data API to fetch videos from a specific YouTube channel based on a minimum duration and number of videos. It also fetches transcripts for these videos using the `youtube-transcript-api`.

## Installation

To run this code, you need to install the following Python libraries:

```bash
!pip install beautifulsoup4 requests isodate youtube-transcript-api
```

## Usage

### Get Channel ID by Username

This function fetches the channel ID for a given username.

```python
def get_channel_id_by_username(username, api_key):
    url = "https://youtube-v31.p.rapidapi.com/channels"
    querystring = {"part": "snippet,contentDetails,statistics", "forUsername": username}
    headers = {
        "X-RapidAPI-Key": api_key,
        "X-RapidAPI-Host": "youtube-v31.p.rapidapi.com"
    }
    response = requests.get(url, headers=headers, params=querystring)
    if response.status_code == 200:
        items = response.json().get('items', [])
        if items:
            return items[0]['id']
        else:
            print("Channel not found.")
            return None
    else:
        print("Failed to fetch data:", response.status_code, response.text)
        return None
```

### Get YouTube Videos

This function fetches videos from a YouTube channel based on minimum duration and number of videos.

```python
def get_youtube_videos(channel_id, api_key, min_minutes, count):
    url = "https://youtube-v31.p.rapidapi.com/search"
    min_seconds = min_minutes * 60
    all_videos = []
    next_page_token = None

    while len(all_videos) < count:
        querystring = {
            "channelId": channel_id,
            "part": "snippet",
            "order": "date",
            "maxResults": "50",
            "pageToken": next_page_token
        }
        headers = {
            "X-RapidAPI-Key": api_key,
            "X-RapidAPI-Host": "youtube-v31.p.rapidapi.com"
        }
        response = requests.get(url, headers=headers, params=querystring)
        if response.status_code == 200:
            videos = response.json().get('items', [])
            video_ids = [video['id']['videoId'] for video in videos if 'videoId' in video['id']]
            video_details = get_video_details(video_ids, api_key)
            filtered_videos = [video for video in video_details if video['duration_seconds'] >= min_seconds]
            all_videos.extend(filtered_videos)
            next_page_token = response.json().get('nextPageToken')
            if not next_page_token:
                break
        else:
            print("Failed to fetch data:", response.status_code, response.text)
            break
    return all_videos[:count]
```

### Get Video Details

This function fetches detailed information including duration for a list of video IDs.

```python
def get_video_details(video_ids, api_key):
    url = "https://youtube-v31.p.rapidapi.com/videos"
    video_details = []
    for i in range(0, len(video_ids), 50):
        querystring = {
            "part": "contentDetails,snippet",
            "id": ','.join(video_ids[i:i+50])
        }
        headers = {
            "X-RapidAPI-Key": api_key,
            "X-RapidAPI-Host": "youtube-v31.p.rapidapi.com"
        }
        response = requests.get(url, headers=headers, params=querystring)
        if response.status_code == 200:
            videos = response.json().get('items', [])
            for video in videos:
                duration = isodate.parse_duration(video['contentDetails']['duration'])
                duration_seconds = duration.total_seconds()
                video_details.append({
                    'title': video['snippet']['title'],
                    'url': f"https://www.youtube.com/watch?v={video['id']}",
                    'duration': video['contentDetails']['duration'],
                    'duration_seconds': duration_seconds
                })
        else:
            print("Failed to fetch video details:", response.status_code, response.text)
            break
    return video_details
```

### Get Transcripts

This function fetches transcripts for a list of video URLs.

```python
def get_transcripts(video_urls):
    transcripts = {}
    for video_url in video_urls:
        video_id = video_url.split('=')[-1]
        try:
            transcript_list = YouTubeTranscriptApi.get_transcript(video_id)
            transcript = "\n".join([entry['text'] for entry in transcript_list])
            transcripts[video_url] = transcript
        except Exception as e:
            transcripts[video_url] = f"Transcript not available: {e}"
    return transcripts
```

### Example Usage

Replace the placeholder `api_key` with your actual RapidAPI key.

```python
api_key = "Enter your RapidAPI key"
channel_user_name = 'PowerfulJRE'
channel_id = get_channel_id_by_username(channel_user_name, api_key)

min_minutes = 10  # Minimum duration in minutes
count = 2  # Number of videos to return
videos = get_youtube_videos(channel_id, api_key, min_minutes, count)
print(f"Found {len(videos)} videos with a minimum duration of {min_minutes} minutes.")
for video in videos:
    print(f"Title: {video['title']}, URL: {video['url']}, Duration: {video['duration']}")

urls = get_video_urls(videos)
transcriptions = get_transcripts(urls)
print(transcriptions)
```

### Dependencies

- `beautifulsoup4`
- `requests`
- `isodate`
- `youtube-transcript-api`
- `Rapid-api-ket`


### Contributing

Contributions are welcome. Please open an issue to discuss what you would like to change.

### Contact

For any issues, please contact [Mathavan S G](aimathavan14@gmail.com).
