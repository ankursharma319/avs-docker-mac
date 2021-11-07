# Instructions for running Alexa AVS in a docker on mac

## docker pulseaudio

```
paplay /Users/ankurs4/Projects/audio/minimp3/alexa.wav
pulseaudio --load=module-native-protocol-tcp --exit-idle-time=-1 --daemon
pulseaudio --check -v
pacmd -h
pacmd play-file ~/Downloads/crowd.wav 1__Channel_2

docker run -it -e PULSE_SERVER=docker.for.mac.localhost -v ~/.config/pulse:/home/pulseaudio/.config/pulse --entrypoint speaker-test --rm jess/pulseaudio -c 2 -l 1 -t wav

docker build ./ --file Dockerfile-pulseaudio --tag ankurs4/pulseaudio:0.1

docker run -it -e PULSE_SERVER=docker.for.mac.localhost \
-v ~/.config/pulse:/root/.config/pulse \
--name pulseaudio-test --privileged --detach ankurs4/pulseaudio:0.1 bash

speaker-test -c 2 -l 1 -t wav
```

## docker avs-test

clone device sdk with sample app (+phonecallcontrolller if needed) to my_project/

```
docker-compose up --detach
```

https://developer.amazon.com/en-GB/docs/alexa/avs-device-sdk/ubuntu.html

```
# Need to install curl and nghttp2 manually from source

##### nghttp2 ######
#is simply ./configure and make and make install and ldconfig

####### for curl #####
#cd $HOME/my_project/third-party
#wget http://curl.haxx.se/download/curl-7.54.0.tar.bz2
#tar -xvjf curl-7.54.0.tar.bz2
#cd curl-7.54.0
#./configure --with-nghttp2=/usr/local --with-ssl
#make
#sudo make install
#sudo ldconfig
# might also have to add /usr/local/bin/ to path
```

```
cmake $HOME/my_project/source/avs-device-sdk \
 -DGSTREAMER_MEDIA_PLAYER=ON \
 -DPORTAUDIO=ON \
 -DPORTAUDIO_LIB_PATH=$HOME/my_project/third-party/portaudio/lib/.libs/libportaudio.a \
 -DPORTAUDIO_INCLUDE_DIR=$HOME/my_project/third-party/portaudio/include \
 -DCMAKE_BUILD_TYPE=DEBUG \
 -DPCC=ON

 -DCMAKE_BUILD_TYPE=DEBUG \
 -DCMAKE_BUILD_TYPE=RELEASE \
 MINSIZEREL

bash genConfig.sh config.json 12345 \
 $HOME/my_project/build/Integration/database \
 $HOME/my_project/source/avs-device-sdk \
 $HOME/my_project/build/Integration/AlexaClientSDKConfig.json \
 -DSDK_CONFIG_MANUFACTURER_NAME="Manufacturer" \
 -DSDK_CONFIG_DEVICE_DESCRIPTION="Description"

cd ~/my_project/build
./SampleApp/src/SampleApp ./Integration/AlexaClientSDKConfig.json DEBUG9

size --format=Berkeley --totals SampleApp/src/SampleApp
ldd SampleApp/src/SampleApp

valgrind --tool=massif --pages-as-heap=yes --time-unit=B --stacks=yes --depth=1 ./SampleApp/src/SampleApp ./Integration/AlexaClientSDKConfig.json DEBUG9
ms_print massif.out

top
free -m
smem -k
```

## Testing mem usage of different builds

DEBUG BUILD WITH MLOCKALL

When speaking,
```
PID User     Command                         Swap      USS      PSS      RSS 
 4887 root     ./SampleApp/src/SampleApp .        0   758.7M   759.4M   761.1M 
```

When idle,
```
 4887 root     ./SampleApp/src/SampleApp .        0   672.3M   673.1M   674.8M
```

DEBUG BUILD WITHOUT MLOCKALL
```
 6308 root     ./SampleApp/src/SampleApp .        0    94.1M    94.8M    96.5M 
```

MINSIZEREL BUILD WITHOUT MLOCKALL
```
8646 root     ./SampleApp/src/SampleApp .        0    68.3M    69.0M    70.7M 
```

MINSIZEREL WITH MLOCKALL
```
 9204 root     ./SampleApp/src/SampleApp .        0   752.1M   752.9M   754.8M 
```
