# Workshop for KVS

패키지 목록을 업데이트하고 SDK를 구축하는 데 필요한 라이브러리를 설치합니다.

```c
sudo apt update
sudo apt install -y cmake gstreamer1.0-plugins-bad gstreamer1.0-plugins-good gstreamer1.0-plugins-ugly gstreamer1.0-tools libgstreamer-plugins-base1.0-dev
```

## SDK 

#### 다운로드

아래와 같이 SDK를 다운로드합니다. 

```c
cd ~
git clone https://github.com/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp.git
```

#### 빌드

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

## 테스트

아래 테스트를 진행하기 위해서는 아래와 같은 credential을 환경변수로 등록하여야 합니다. [임시 Credential 생성](https://github.com/kyopark2014/aws-security-token-service/blob/main/credential-using-aws-cli.md#iam-role-%EC%83%9D%EC%84%B1)을 참조하여 아래 값들을 채웁니다. 

```c
export AWS_DEFAULT_REGION=ap-northeast-1
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export AWS_SESSION_TOKEN=
```

```c
cd ~
wget https://sehyul-us-east-1.s3.amazonaws.com/samples/sample.mp4
```

```c
export GST_PLUGIN_PATH=$HOME/amazon-kinesis-video-streams-producer-sdk-cpp/build
export LD_LIBRARY_PATH=$HOME/amazon-kinesis-video-streams-producer-sdk-cpp/open-source/local/lib

cd ~/amazon-kinesis-video-streams-producer-sdk-cpp/build
while true; do ./kvs_gstreamer_file_uploader_sample kvs-workshop-stream ~/sample.mp4 $(date +%s) audio-video && sleep 10s; done

```




