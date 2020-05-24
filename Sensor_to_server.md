Send data from sensor to server
======================


### Sensor's physical line connecting

- ESP32_CAM(ov2640) to esp32 board
<img src="https://github.com/jihwahn1018/Capston_Design1_Projects/blob/master/images/esp32cam_line.jpg" width="400">

- 초음파 센서(HC-SR04) to esp32 board
<img src="https://github.com/jihwahn1018/Capston_Design1_Projects/blob/master/images/hc_line.jpg" width="400">

### Sensor code 

- ESP32_CAM code(.ino 파일, h파일 생략)
기존 Arduino의 예제 ESP32->Camera->CameraWebSever 코드 응용
(기존 예시 코드 : 동일한 wifi 공유기 내에서만 접근 가능 -> 변형된 코드 : Amazon EC2 server을 이용해 어디서든 접근 가능  )

<pre>
<code>
#include "esp_camera.h" // 카메라 사용을 위한 기본 헤더 파일
#include <WiFi.h> //wifi 연결을 위한 헤더 파일
#include <ArduinoWebsockets.h> // Amazon EC2에 열어둔 서버의 소켓으로 보내기 위한 헤더 파일

// Select camera model
#define CAMERA_MODEL_AI_THINKER

#include "camera_pins.h"

const char* ssid = "****";
const char* password = "****";
const char* websocket_server_host = "****"; //데이터를 보낼 서버의 
const uint16_t websocket_server_port = ****; // 데이터를 보내 서버의 소켓번호

using namespace websockets;
WebsocketsClient client;

void setup() {
  Serial.begin(115200);
  Serial.setDebugOutput(true);
  Serial.println();

  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 10000000;
  config.pixel_format = PIXFORMAT_JPEG;
  //init with high specs to pre-allocate larger buffers
  if(psramFound()){
    config.frame_size = FRAMESIZE_VGA;
    config.jpeg_quality = 40;
    config.fb_count = 2;
  } else {
    config.frame_size = FRAMESIZE_SVGA;
    config.jpeg_quality = 12;
    config.fb_count = 1;
  }


  // camera init
  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x", err);
    return;
  }

 

  WiFi.begin(ssid, password); //Wifi를 연결해준다.

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");

  Serial.print("Camera Ready! Use 'http://");
  Serial.print(WiFi.localIP());
  Serial.println("' to connect");
  
  //Amazon EC2 서버와 연결해 준다.
  while(!client.connect(websocket_server_host, websocket_server_port, "/")){
    delay(500);
    Serial.print(".");
  }
  Serial.println("Websocket Connected!");
}

void loop() {

  //카메라가 제대로 작동 하는지 확인
  camera_fb_t *fb = esp_camera_fb_get();
  if(!fb){
    Serial.println("Camera capture failed");
    esp_camera_fb_return(fb);
    return;
  }

  //파일 형식이 JPEG인지 확인
  if(fb->format != PIXFORMAT_JPEG){
    Serial.println("Non-JPEG data not implemented");
    return;
  }
  //Binary 파일 형태로 데이터 전송
  client.sendBinary((const char*) fb->buf, fb->len);
  esp_camera_fb_return(fb);
}

  
</code>
</pre>
