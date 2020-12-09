---
title:  "Python stt code(with file)"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - STT
  
tags:
  - STT
  - python
  - wav
  - mp3
  - raw
  
last_modified_at: 2020-12-09
---

google cloud stt api를 사용하는 코드를 참조했다. 참고로 출처는 [구글 stt 빠른시작 document](https://cloud.google.com/speech-to-text/docs/quickstart-client-libraries?hl=ko)이다.

우선 빠른 시작 페이지에 나와있는 코드는 아래와 같다.

```python
import io
import os

# Imports the Google Cloud client library
from google.cloud import speech

# Instantiates a client
client = speech.SpeechClient()

# The name of the audio file to transcribe
file_name = os.path.join(os.path.dirname(__file__), "resources", "audio.raw")

# Loads the audio into memory
with io.open(file_name, "rb") as audio_file:
    content = audio_file.read()
    audio = speech.RecognitionAudio(content=content)

config = speech.RecognitionConfig(
    encoding=speech.RecognitionConfig.AudioEncoding.LINEAR16,
    sample_rate_hertz=16000,
    language_code="en-US",
)

# Detects speech in the audio file
response = client.recognize(config=config, audio=audio)

for result in response.results:
    print("Transcript: {}".format(result.alternatives[0].transcript))
```

하지만 단순히 위와 같이 코드를 돌리면 여러 문제가 발생할 수 있다.


1. google cloud api를 사용하려면 서비스 계정 인증 정보(json파일 형태)가 필요한데, 위의 코드에서는 따로 지정해주는 부분이 없다.
2. `sample_rate_hertz`가 위의 sample에서는 16000인데, 오디오 파일에 따라 저 rate가 달라질 수 있다.
3. language_code는 현재 영어로 되어 있어서 한글 오디오 파일을 넣을 경우 제대로 원하는 값이 나오지 않을 수 있다.
4. 오디오 파일에 따라 `speech.RecognitionConfig`안에 channel 파라미터를 설정해야 할 수 있다.

각각에 대한 해결 방법은 다음과 같다.

1. 위에 첨부한 빠른 시작 페이지에서처럼 `export GOOGLE_APPLICATION_CREDENTIALS="[PATH]"`커맨드를 통해서 직접 커맨드를 통해 인증 정보를 넣거나 코드 상으로 명시해준다. 나는 매번 커맨드를 통해 설정하기가 귀
  귀찮아서 코드 상으로 명시했다.(참고로 json파일-인증정보 는 미리 google cloud console에서 다운받아서 로컬에 저장해놔야 한다.)
2. rate가 다른 경우는 코드를 실행시키고 나서 확인 후 고칠 수 있다. 이게 무슨 말이냐면
  ![스크린샷 2020-12-09 오후 2 11 20](https://user-images.githubusercontent.com/41438361/101587916-8c302380-3a28-11eb-9fd8-eff12d221eef.png)
  
  위의 스샷처럼 "sample_rate_hertz (48000) in RecognitionConfig must either be omitted or match the value in the WAV header ( 44100)."과 같은 에러 메세지가 뜨는 경우에는 오디오 파일의 
  rate_hertz가 44100인데 내가 코드 상으로 48000으로 설정했을 경우이다. 그래서 에러 메세지를 보고 `sample_rate_hertz`의 값을 조정해주면 된다.
3. language_code는 한국어일 경우 "ko-KR"로 바꿔주면 된다.
4. 이 경우도 에러 메세지를 통해 확인 후 수정할 수 있다.
  ![스크린샷 2020-12-09 오후 2 15 55](https://user-images.githubusercontent.com/41438361/101588155-15475a80-3a29-11eb-8e93-fe6c032dab50.png)
  
  위와 같이 메세지가 뜨면 코드 상에서 내가 single channel을 사용한다고(위에 첨부한 코드 같은 경우이다) 해놨는데, 오디오 파일에서 두 개의 채널을 요구하는 경우이다. 이럴 경우 `speech.RecognitionConfig`안에 
  channel 파라미터의 값을 조정해주면 된다. 아래에 내가 수정한 코드처럼 수정하면 된다.
  
  
내가 최종적으로 수정한 코드는 다음과 같다.

```python
import io
import os

# Imports the Google Cloud client library
from google.cloud import speech
from google.oauth2.service_account import Credentials

# Instantiates a client. Added credential information
creds = Credentials.from_service_account_file("credential_file_location.json") 
client = speech.SpeechClient(credentials=creds)

# The name of the audio file to transcribe
file_name = os.path.join(os.path.dirname(__file__), "audio_file_name.wav")

# Loads the audio into memory
with io.open(file_name, "rb") as audio_file:
    content = audio_file.read()
    audio = speech.RecognitionAudio(content=content)

config = speech.RecognitionConfig(
    encoding=speech.RecognitionConfig.AudioEncoding.LINEAR16,
    sample_rate_hertz=44100,
    language_code="ko-KR",
    audio_channel_count=2,
)

# Detects speech in the audio file
response = client.recognize(config=config, audio=audio)

for result in response.results:
    print("Transcript: {}".format(result.alternatives[0].transcript))
```
  

  
