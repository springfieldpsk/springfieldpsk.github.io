---
title: esp8266Mqtt
date: 2022-04-10 21:46:35
tags:
---
## 思路

esp8266连接至本地wifi后，通过mqtt链接至服务器，订阅指定mqtt主题，获取指定主题的mqtt消息，并通过u8g2库显示在屏幕上

## 流程

1. 在esp8266上注册WIFI信息与mqtt服务器信息
2. 在mqtt服务器上注册esp8266所使用的账号
3. 指定使用的主题并订阅
4. 获取订阅主题的信息并绘制至屏幕

<!-- more -->

## 代码实现

```c
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <Arduino.h>
#include <U8g2lib.h>

#ifdef U8X8_HAVE_HW_SPI
#include <SPI.h>
#endif
#ifdef U8X8_HAVE_HW_I2C
#include <Wire.h>
#endif

//状态define
#define STATE_INIT  0
#define STATE_WIFI_CONNECTING 1
#define STATE_MQTT_CONNECTING 2
#define STATE_WORKING 3
#define STATE_MQTT_ERROR  4
#define STATE_WIFI_ERROR  5

//参数定义
const char* ssid = "";
const char* password = "";
const char* mqttServer = "";
const char* mqttServerId = "";
const int mqttPort = ;
const char* mqttUser = "";
const char* mqttPassword = "";

// wifi client
WiFiClient espClient;
// mqtt client
PubSubClient client(espClient);
// 状态参数
int workstate = STATE_INIT;

// mqtt主题
const char* ReciveTopic="data/#";
const char* UploadTopic="data/UpdateMess";
char* RawText = "Init";

// u8g2接口
U8G2_SSD1306_128X64_NONAME_F_SW_I2C u8g2(U8G2_R0, /* clock=*/ SCL, /* data=*/ SDA, /* reset=*/ U8X8_PIN_NONE);   // All Boards without Reset of the Display

void setup() {
    // 开放串口 波特率为115200
    Serial.begin(115200);    

    // 启动u8g2
    u8g2.begin();
    u8g2.enableUTF8Print();		// enable UTF8 support for the Arduino print() function
}

void loop() {
    
    int ret = 0 ;
    int mqtt_err_times = 0;

    SerialDataHandle();

    switch (workstate){
        // 工作状态为初始化时
        case STATE_INIT:
            // 判断WIFI 与 MQTT信息
            if( ssid == "" || password == ""){
                Serial.println("Missing ssid / password");
                RawText = "Missing ssid / password";
                break;
            }

            if( mqttServer == "" || mqttServerId == "" || mqttUser == "" || mqttPassword == ""){
                Serial.println("Missing mqtt Inf");
                RawText = "Missing mqtt Inf";
                break;
            }
            // 信息无误 转入正在WIFI链接状态
            workstate = STATE_WIFI_CONNECTING;
            break;

        // 工作状态为WIFI链接状态时
        case STATE_WIFI_CONNECTING:
            // 执行WIFI链接函数
            ret = WifiConnect();

            // 链接成功 进入MQTT链接模式
            if( ret == 0 ) {
                workstate = STATE_MQTT_CONNECTING;
            }
            else {
                // 链接失败 转入链接错误状态
                workstate = STATE_WIFI_ERROR;
            }

            break;
        
        // MQTT 链接中
        case STATE_MQTT_CONNECTING:

            // 执行MQTT链接函数
            ret = MqttConnect();

            if(ret == 0){
                
                mqtt_err_times = 0;
                // 启动结束 进入工作状态
                workstate = STATE_WORKING;
            }
            else {

                workstate = STATE_MQTT_ERROR;
            }
            break;
        
        case STATE_WORKING:
            // WIFI 循环 保证链接
            client.loop();

            // 当WIFI链接出现异常时，转异常处理
            if(!client.connected()){
                if(WiFi.status() != WL_CONNECTED){
                    workstate = STATE_WIFI_ERROR;
                }
                else {
                    workstate = STATE_MQTT_ERROR;
                }
            }

            break;

        case STATE_MQTT_ERROR:
            mqtt_err_times ++;

            // 若mqtt无法重连超过5次 则返回WIFI ERROR
            if(mqtt_err_times >=5){
                mqtt_err_times = 0;
                workstate = STATE_WIFI_ERROR;
            }
            
            break;

        case STATE_WIFI_ERROR:
            workstate = STATE_INIT;
            break;
        
        default:
            workstate = STATE_INIT;
            break;
    }
}

int WifiConnect(){
    int retry_times = 0;
    // 根据ssid 与 密码启动WIFI
    WiFi.begin(ssid,password);

    Serial.print("In Connecting\n");
    RawText = "In Connecting...";

    // 链接WIFI 若超过10次 则WIFI TIme OUT 返回WIFI ERROR状态
    while(WiFi.status() != WL_CONNECTED){
        delay(500);

        retry_times++;
        
        if(retry_times >= 10){
            // Serial.println(WiFi.status());
            Serial.print("WifI Connect timeout\n");
            RawText = "WifI Connect timeout";
            return -1;
        }
    }

    Serial.print("WiFI Connected, IP address: ");
    Serial.println(WiFi.localIP());
    RawText = "WiFI Connected";
    return 0;
}

int MqttConnect(){
    int retry_times = 0;

    // 根据 URL 与 Port 设置 MQTT服务器
    client.setServer(mqttServer,mqttPort);

    // 设置MQTT回调函数
    client.setCallback(callback);

    // 启动MQTT链接 无法连接超过10次判定失败 
    while(!client.connected()){
        Serial.println("MQTT Server Connecting......");
        RawText = "MQTT Server Connecting......";
        // 根据 username 与 password 链接mqtt服务器
        if(client.connect(mqttServerId,mqttUser,mqttPassword)){
            Serial.print("connected to ");
            Serial.println(mqttServerId);

            RawText = "connected to ";
            strcmp(RawText,mqttServerId);

            SetSubscribe();
            break;
        }
        else {
            Serial.print("failed with state: ");
            Serial.println(client.state());

            RawText = "failed Connect";

            retry_times ++;
            if(retry_times >= 10){
                Serial.print("MQTT Connect timeout\n");
                RawText = "MQTT Connect timeout";
                
                return -1;
            }
        }
        delay(500);
    }

    return 0;
}

void callback(char *topic,byte* payload, unsigned int length){
    Serial.println("------ New  Message ------");
    Serial.printf("Message arrived in topic: ");
    Serial.printf(topic);
    Serial.printf("Message: ");

    // 将接受的订阅信息转化为字符串
    for(int i = 0 ; i < length ; i ++){
        Serial.print((char)payload[i]);
        RawText[i] = (char)payload[i];
    }
    RawText[length] = '\0';

    Serial.println();
    Serial.println("----- End of Message -----");

}

void PublishBack(const char* Message){
    if(client.publish(UploadTopic,Message)){
        Serial.printf("Upload Message to %s , Message = %s\n",UploadTopic,Message);
    }
}

void SerialDataHandle(){
    // 设置屏幕
    addMessageinScreen(RawText);
}

void addMessageinScreen(char* Message){
    u8g2.setFont(u8g2_font_unifont_t_chinese2);  
    u8g2.setFontDirection(0);
    
    u8g2.clearBuffer();
    u8g2.setCursor(0, 15);
    u8g2.print(Message);
    u8g2.sendBuffer();

    delay(100);
}

void SetSubscribe(){
    // 设置订阅主题
    client.subscribe(ReciveTopic);
    Serial.print("Set subscrbe ");
    Serial.println(ReciveTopic);
}
```