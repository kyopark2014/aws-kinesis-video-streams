# Workshop for KVS

여기서는 [Amazon Kinesis Video Streams for beginners](https://catalog.us-east-1.prod.workshops.aws/workshops/82de7f30-615c-4adf-b4ae-3438f451735c/ko-KR)의 내용을 기준으로 KVS에 동영상을 업로드하는 동작에 대해 설명합니다. 

## KVS를 위한 환경준비

Cloud9 또는 EC2를 이용하여 IoT device처럼 KVS 동작을 확인할 수 있습니다. 아래의 설명은 편의상 "Ubuntu Server 18.04 LTS"을 기준으로 합니다.

#### 필수 라이브러리 설치 

준비한 Cloud9 또는 EC2에 패키지 목록을 업데이트하고 SDK를 구축하는 데 필요한 라이브러리를 설치합니다.

```c
sudo apt update
sudo apt-get install -y libssl-dev libcurl4-openssl-dev liblog4cplus-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-base-apps gstreamer1.0-plugins-bad gstreamer1.0-plugins-good gstreamer1.0-plugins-ugly gstreamer1.0-tools
```

#### SDK Download

아래와 같이 [Amazon KVS를 위한 SDK](https://github.com/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp)를 다운로드합니다. 여기에는 CPP Producer, GStreamer Plugin와 JNI가 있습니다.

```c
git clone https://github.com/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp.git
```

#### SDK Build

아래와 같이 SDK를 다운로드하고 빌드합니다. 

```c
mkdir -p ~/amazon-kinesis-video-streams-producer-sdk-cpp/build
cd ~/amazon-kinesis-video-streams-producer-sdk-cpp/build
cmake -DBUILD_GSTREAMER_PLUGIN=ON ..
make
```

cmake 미설치로 인한 에러시에는 아래처럼 설치후 재시도합니다.

```c
sudo apt --fix-broken install
sudo apt install cmake
```

#### KVS 설치 

1) [[KVS Console](https://ap-northeast-2.console.aws.amazon.com/kinesisvideo/home?region=ap-northeast-2#/streams)]로 이동하여 [Create video stream]을 선택합니다. 

2) 아래와 같이 [Video stream name]으로 "iot-stream"을 입력하고, [Create video stream]을 선택합니다.  

![noname](https://user-images.githubusercontent.com/52392004/190509015-b616afea-acf4-4a2f-a010-173a0b2e7b6b.png)


## KVS에 파일업로드 시험

KVS에 파일업로드를 위해서는 인증과정이 필요합니다. 여기서는 Temparary Credential을 생성하여, 환경변수로 등록하여 사용합니다. [Temparary Credential 생성](https://github.com/kyopark2014/aws-security-token-service/blob/main/credential-using-aws-cli.md#iam-role-%EC%83%9D%EC%84%B1)을 참조하여 아래 값들을 채웁니다. 

1) Temparary Credential 생성

아래 명령어를 AWS CLI와 credential이 설치된 PC에서 실행합니다. PC에 설치되지 않았다면, "aws configure" 명령어로 credential을 입력후 수행합니다.

```c
aws sts get-session-token
```

이때의 결과의 예는 아래와 같습니다. 

```c
{
    "Credentials": {
        "AccessKeyId": "SAMPLEKIXN5TEZDODGH5",
        "SecretAccessKey": "SampleVMcYYwzHhUPW8KsH2vr2CYQHvAXXECnM9w",
        "SessionToken": "SampleJpZ2luX2VjEEUaDmFwLW5vcnRoZWFzdC0yIkcwRQIhAMzoOicBxEi23325b7uX4VlYanygRPXVMpVZkpO/oL3LAiBX/G/TU+hKlI7sUUvgq7S22qEpFZjteyCAv353S51Ynir0AQje//////////8BEAEaDDY3NzE0Njc1MDgyMiIM3XXl03EGf1PmjbCfKsgB21sUTdSr6dhuz7ZDW66jOz/R6x+Vt/N6aSamplee6kFUCrl99M/DZGQpcxaMlm83exnFTNn/bbW8imp5hY0MTo97/lLd4BZby6OKMZd8zoLIe8moy7g2QK/iZydTw8WoWQCvaT5KJpwFfdBS/TsaJ+urM3B3niwvNZyLaRHRnI+2Du1kQWHUJ7SZ/7uUX1QlLLwVVvlfukqDLWIbz6NHMgKm7H9iGqPhGFP8jSCBephHmphrR3ONu55AbCm+9/eeTRDHuVvBTUYwgpyOmQY6mAGS21Tm2k/3OKfANaL1vyjSampleUF6f64ZkqN3E+27qrhTNtooxyH7v2/mxWiLeLobtUoyDR2V74Y36Py4mRVTgIO5ItIN9LpQQmTwUbTqtMeaPDBCf3hn+eUL+8Y+5xGw8xtr9eOXURwCexupKrgYyo5VpBlckrJKK03uFY3IIm+H8sFeR00twwfZvQ5aMDzCCEAQGbeQJg==",
        "Expiration": "2022-09-16T08:41:38+00:00"
    }
}
```

Temperary credential은 사용하려는 client에 아래와 같이 환경변수로 등록합니다. 이때, 사용하려는 Default Region도 아래처럼 설정하여야 합니다. 

```c
export AWS_DEFAULT_REGION=ap-northeast-2
export AWS_ACCESS_KEY_ID=SAMPLEKIXN5TEZDODGH5
export AWS_SECRET_ACCESS_KEY=SampleVMcYYwzHhUPW8KsH2vr2CYQHvAXXECnM9w
export AWS_SESSION_TOKEN=SampleJpZ2luX2VjEEUaDmFwLW5vcnRoZWFzdC0yIkcwRQIhAMzoOicBxEi23325b7uX4VlYanygRPXVMpVZkpO/oL3LAiBX/G/TU+hKlI7sUUvgq7S22qEpFZjteyCAv353S51Ynir0AQje//////////8BEAEaDDY3NzE0Njc1MDgyMiIM3XXl03EGf1PmjbCfKsgB21sUTdSr6dhuz7ZDW66jOz/R6x+Vt/N6aSamplee6kFUCrl99M/DZGQpcxaMlm83exnFTNn/bbW8imp5hY0MTo97/lLd4BZby6OKMZd8zoLIe8moy7g2QK/iZydTw8WoWQCvaT5KJpwFfdBS/TsaJ+urM3B3niwvNZyLaRHRnI+2Du1kQWHUJ7SZ/7uUX1QlLLwVVvlfukqDLWIbz6NHMgKm7H9iGqPhGFP8jSCBephHmphrR3ONu55AbCm+9/eeTRDHuVvBTUYwgpyOmQY6mAGS21Tm2k/3OKfANaL1vyjSampleUF6f64ZkqN3E+27qrhTNtooxyH7v2/mxWiLeLobtUoyDR2V74Y36Py4mRVTgIO5ItIN9LpQQmTwUbTqtMeaPDBCf3hn+eUL+8Y+5xGw8xtr9eOXURwCexupKrgYyo5VpBlckrJKK03uFY3IIm+H8sFeR00twwfZvQ5aMDzCCEAQGbeQJg==
```

2) 셈플동영상 다운로드 

아래처럼 [github](https://github.com/kyopark2014/aws-kinesis-video-streams)의 sample.mp4을 다운로드 합니다.

```c
cd ~
git clone https://github.com/kyopark2014/aws-kinesis-video-streams
ls -l $HOME/aws-kinesis-video-streams/samples
```

3) SDK를 위한 환경변수를 등록합니다. 

```c
export GST_PLUGIN_PATH=$HOME/amazon-kinesis-video-streams-producer-sdk-cpp/build
export LD_LIBRARY_PATH=$HOME/amazon-kinesis-video-streams-producer-sdk-cpp/open-source/local/lib
```

4) 동영상을 "iot-stream"으로 반복적으로 전송합니다. 

```c
cd ~/amazon-kinesis-video-streams-producer-sdk-cpp/build
while true; do ./kvs_gstreamer_file_uploader_sample iot-stream $HOME/aws-kinesis-video-streams/samples/sample.mp4 $(date +%s) audio-video && sleep 10s; done
```

상기 명령어 실행시 아래와 같이 동작합니다. 

```c
[DEBUG] [15-09-2022 21:27:04:455.062 GMT] fragmentAckReceivedHandler invoked
[DEBUG] [15-09-2022 21:27:04:553.098 GMT] postWriteCallback(): Curl post body write function for stream with handle: iot-stream and upload handle: 0 returned: {"EventType":"PERSISTED","FragmentTimecode":1663277247152,"FragmentNumber":"91343852333181432511524306419788296366724229586"}

[DEBUG] [15-09-2022 21:27:04:563.256 GMT] fragmentAckReceivedHandler invoked
[INFO ] [15-09-2022 21:27:04:563.315 GMT] getStreamData(): Indicating an EOS after last persisted ACK is received for stream upload handle 0
[DEBUG] [15-09-2022 21:27:04:563.333 GMT] streamClosedHandler invoked for upload handle: 0
[DEBUG] [15-09-2022 21:27:04:563.346 GMT] Reported streamClosed callback for stream handle 94222702636784. Upload handle 0
[INFO ] [15-09-2022 21:27:04:563.387 GMT] Sending eos
[INFO ] [15-09-2022 21:27:04:563.756 GMT] postReadCallback(): Reported end-of-stream for stream iot-stream. Upload handle: 0
[DEBUG] [15-09-2022 21:27:04:563.878 GMT] postReadCallback(): Wrote 0 bytes to Kinesis Video. Upload stream handle: 0
[INFO ] [15-09-2022 21:27:04:564.156 GMT] Freeing Kinesis Video Stream iot-stream
[INFO ] [15-09-2022 21:27:04:564.216 GMT] freeKinesisVideoStream(): Freeing Kinesis Video stream.
[DEBUG] [15-09-2022 21:27:04:564.246 GMT] curlApiCallbacksShutdownActiveRequests(): pActiveRequests hashtable is empty
[INFO ] [15-09-2022 21:27:07:665.467 GMT] freeKinesisVideoClient(): Freeing Kinesis Video Client
[DEBUG] [15-09-2022 21:27:07:665.527 GMT] curlApiCallbacksShutdownActiveRequests(): pActiveRequests hashtable is empty
[DEBUG] [15-09-2022 21:27:08:066.250 GMT] freeKinesisVideoClientInternal(): Total allocated memory 0
[WARN ] [15-09-2022 21:27:08:066.328 GMT] curlApiCallbacksShutdown(): curlApiCallbacksShutdown called when already in progress of shutting down
[INFO] kvs_gstreamer_file_uploader_sample: Persisted successfully. File: /home/ubuntu/aws-kinesis-video-streams/samples/sample.mp4
```

5) 전송된것을 KVS에서 확인합니다. 

[[KVS streams console](https://ap-northeast-2.console.aws.amazon.com/kinesisvideo/home?region=ap-northeast-2#/streams)]로 이동한후, [KVS 설치](https://github.com/kyopark2014/aws-kinesis-video-streams/blob/main/workshop.md#kvs-%EC%84%A4%EC%B9%98)에서 생성한 "iot-stream"를 선택합니다. 

이후, 아래와 같이 [Media playback]을 선택하면, 업로드 되고 있는 동영상을 확인할 수 있습니다. 

![noname](https://user-images.githubusercontent.com/52392004/190512431-4d0101f6-4552-42ba-9cf0-649102ca67d4.png)


## Reference

[Amazon Kinesis Video Streams for beginners](https://catalog.us-east-1.prod.workshops.aws/workshops/82de7f30-615c-4adf-b4ae-3438f451735c/ko-KR)




