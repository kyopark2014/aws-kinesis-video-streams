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
git clone --recursive https://github.com/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp.git
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

```c
cd ~
wget https://sehyul-us-east-1.s3.amazonaws.com/samples/sample.mp4
```

