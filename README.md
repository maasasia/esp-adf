# Forked from espressif/esp-adf
original document is here. https://github.com/espressif/esp-adf

## Install guide
1. Clone this repository
```
git clone -b v2.4-modify1 --recursive git@github.com:maasasia/esp-adf.git
```
2. Set the $ESP_ADF path
```
cd esp-adf
export ADF_PATH=$PWD
```
3. Process submodule
```
git submodule update --init --recursive
```

## Why?

### 아래와 같은 작업이 필요한데 매번 하기 귀찮으니까 fork 떠서 만듦

1.
esp-adf 를 사용하면, 아래의 에러를 만나게 될 것입니다.

In file included from /Users/yohannassouline/esp/esp-adf/components/bluetooth_service/hfp_stream.c:32: /Users/yohannassouline/esp/esp-adf/components/bluetooth_service/include/hfp_stream.h:36:10: fatal error: esp_hf_client_api.h: No such file or directory #include "esp_hf_client_api.h" ^~~~~~~~~~~~~~~~~~~~~ compilation terminated.

해당 이슈는 esp-adf 에러로, esp-adf/components/bluetooth_service/CMakeLists.txt 의 7번째 줄 set(COMPONENT_SRCS ./bluetooth_service.c ./bt_keycontrol.c ./a2dp_stream.c ./hfp_stream.c) 를 주석 처리 하면 돌아갑니다.


2.
sound_manager.cpp 쪽에서 소리 나오는 부분에서, 작동을 조금 더 예측 가능하도록 하기 위해, i2s_stream 이 pause 되지 않도록 하게 해서 사용 중. esp-adf 의 i2s_stream.c 에서 304~306 번째 line 을 주석 처리.
```
if (state == AEL_STATE_RUNNING) {
   audio_element_pause(i2s_stream);
}
```

3.
재생되는 소리들이 저장되는 파티션명을 spiffs 에서 sound 로 변경하면서, adf 코드에서 하드코딩 되어있는 것과 충돌이 있음. spiffs_stream.c 파일의 95번째 line을 변경하면 됨.
```
char *path = strstr(uri, "/spiffs");
```
를
```
char *path = uri;
```
로 변경

4.
adf의 수정을 통해 i2s dac의 설정을 한번만 진행하도록 수정. adf를 변경하지 않아도 작동은 하지만, 불안정한 작동의 원인이 될 수 있음. esp-adf/components/audio_stream/i2s_stream.c 의 393번째 줄
```
i2s_set_dac_mode(I2S_DAC_CHANNEL_BOTH_EN);
```
를
```
i2s_set_dac_mode(I2S_DAC_CHANNEL_RIGHT_EN);
```
로 변경