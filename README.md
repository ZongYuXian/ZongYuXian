- 👋 Hi, I’m @ZongYuXian
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

#include <WiFi.h>
#include <PubSubClient.h>
#include <ESP32Servo.h>

// 舵机的最大角度和最小角度
int servoMax = 360;
int servoMin = 0;
//const int angleStep = 30;
// WiFi连接信息
const char* ssid = "TP-LINK_Zong";
const char* password = "zongyuxian";

// MQTT服务器信息
const char* mqtt_server = "192.168.0.102";
const int mqtt_port = 1883;
const char* mqtt_username = "admin";
const char* mqtt_password = "zyxblr3137";




// 上次动作的时间戳
unsigned long lastActionTime = 0;
const unsigned long actionInterval = 2000; // 动作间隔时间（毫秒）



// 创建一个Servo对象
Servo servo1;
Servo servo2;
// 舵机引脚
const int servoPin1 = 18;
const int servoPin2 = 15;
bool isButtonPressed = false;
bool isRotating = false;
//########################################################################################################################################################################################
int servo2Angle = 0;  // 记录舵机2的角度
//int servo2Direction = 1;  // 记录舵机2的方向，1为正转，-1为反转
//unsigned long servo2RotationDuration = 0;
int servoPosition = 0;  // 舵机当前位置
volatile bool stopRotation = false;  // 标记用于中断停止旋转
bool toggleRotation = false;  // 标记用于舵机来回旋转


// 创建一个WiFi客户端对象
WiFiClient wifiClient;

// 创建一个MQTT客户端对象
PubSubClient client(wifiClient);

// 连接WiFi网络
void connectWiFi() {
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");
}

// 连接MQTT服务器
void connectMQTT() {
  client.setServer(mqtt_server, mqtt_port);
  while (!client.connected()) {
    Serial.println("Connecting to MQTT...");
    if (client.connect("ESP32Client", mqtt_username, mqtt_password)) {
      Serial.println("Connected to MQTT");
    } else {
      Serial.print("MQTT connection failed, rc=");
      Serial.print(client.state());
      Serial.println(" retrying in 5 seconds");
      delay(5000);
    }
  }
}




//################多余添加的###################################################################################################################################################################

void IRAM_ATTR stop() {
  stopRotation = true;  // 触发中断时，设置停止标志位为true
}

//############################################################################################################################################################################################



// 接收MQTT消息的回调函数
void callback(char* topic, byte* payload, unsigned int length) {
  // 将接收到的消息转换为字符串
  String message = "";
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];



//   // 发布舵机的状态
  if (message == "0" || message == "1" || message == "2" || message == "3" || message == "4") {
    client.publish("servo/state", message.c_str());
  }
  // 发布舵机的可用性
  client.publish("homeassistant/switch/servo/availability", "online");
  }

//   // 根据接收到的消息控制舵机
  if (message == "0") {
    // servo1.attach(14);  // 连接舵机到引脚2
    // servo1.write(180);  // 旋转舵机到180度
    // delay(100);  // 延迟1秒，让舵机旋转到指定角度
    // servo1.detach();  // 断开舵机与引脚的连接，停止舵机
    // lastActionTime = millis();
//第二段代码可以控制一直旋转###################################################################################
      if (!isRotating) {        //###########################################################################
         servo1.detach();  // 断开舵机与引脚的连接，停止舵机
        // servo1.attach(14);  // 连接舵机到引脚14###############################################################
        // servo1.write(90);  // 旋转舵机到30度##################################################################
         isRotating = true; //################################################################################
//第二段代码可以控制一直旋转###################################################################################
    } else {//#############################################################################################
        delay(10);
        servo1.attach(18);  // 连接舵机到引脚14##############################################################
        servo1.write(360);  // 旋转舵机到30度################################################################
        isRotating = false;//#################################################################################
        delay(10);
    }//##########################################################################################################
    // delay(500);  // 延迟500毫秒，让舵机旋转到指定角度或停止旋转###############################################
    // lastActionTime = millis();############################################################################
//第二段代码可以控制一直旋转###################################################################################



  } else if (message == "1") {
    servo1.attach(18);  // 连接舵机到引脚2
    servo1.writeMicroseconds(1000);
    servo1.write(0);  // 旋转舵机到0度
    delay(10);  // 延迟1秒，让舵机旋转到指定角度
    servo1.detach();  // 断开舵机与引脚的连接，停止舵机
    lastActionTime = millis();
  }

  
//   // 根据接收到的消息控制舵机2
   else if (message == "2") {


    //  servo2.attach(15);  // 将舵机连接到引脚9
    //  servo2.write(0);   // 将舵机角度设置为0度
    //  delay(1000);      // 停顿1秒钟
    //  servo2.write(90);   // 将舵机角度设置为0度
    //  servo2.detach();  // 断开舵机与引脚的连接，停止舵机




      stopRotation = false;  // 重置停止标志位
      if (servoPosition == 0) {
        servo2.attach(15);  // 将舵机连接到引脚9
        for (int angle = 0; angle <= 90; angle++) {
          if (stopRotation) {
            break;  // 如果标志位为true，立即跳出循环
          }
          servo2.write(angle);  // 将舵机角度逐渐增加
          delay(10);  // 增加延迟以控制舵机旋转速度
        }
        servoPosition = 90;  // 更新舵机位置
        servo2.detach();  // 断开舵机与引脚的连接，停止舵机
      } else if (servoPosition == 90) {
        servo2.attach(15);  // 将舵机连接到引脚9
        for (int angle = 90; angle <= 180; angle++) {
          if (stopRotation) {
            break;  // 如果标志位为true，立即跳出循环
          }
          servo2.write(angle);  // 将舵机角度逐渐增加
          delay(10);  // 增加延迟以控制舵机旋转速度
        }
        servoPosition = 180;  // 更新舵机位置
        servo2.detach();  // 断开舵机与引脚的连接，停止舵机
      }



// //#########################################################################################################
 } else if (message == "3") {


    //  servo2.attach(15);  // 将舵机连接到引脚9
    //  servo2.write(90);   // 将舵机角度设置为0度
    //  delay(1000);      // 停顿1秒钟
    //  servo2.write(180);   // 将舵机角度设置为0度
    //  servo2.detach();  // 断开舵机与引脚的连接，停止舵机


      stopRotation = false;  // 重置停止标志位
      if (servoPosition == 180) {
        servo2.attach(15);  // 将舵机连接到引脚9
        for (int angle = 180; angle >= 90; angle--) {
          if (stopRotation) {
            break;  // 如果标志位为true，立即跳出循环
          }
          servo2.write(angle);  // 将舵机角度逐渐减小
          delay(10);  // 增加延迟以控制舵机旋转速度
        }
        servoPosition = 90;  // 更新舵机位置
        servo2.detach();  // 断开舵机与引脚的连接，停止舵机
      } else if (servoPosition == 90) {
        servo2.attach(15);  // 将舵机连接到引脚9
        for (int angle = 90; angle >= 0; angle--) {
          if (stopRotation) {
            break;  // 如果标志位为true，立即跳出循环
          }
          servo2.write(angle);  // 将舵机角度逐渐减小
          delay(10);  // 增加延迟以控制舵机旋转速度
        }
        servoPosition = 0;  // 更新舵机位置
        servo2.detach();  // 断开舵机与引脚的连接，停止舵机
      }




   }
  else if (message == "4") {
          servo1.attach(18);  // 连接舵机到引脚2
          servo1.write(180);  // 旋转舵机到180度
          delay(5000);  // 延迟1秒，让舵机旋转到指定角度
          servo1.detach();  // 断开舵机与引脚的连接，停止舵机
          lastActionTime = millis();
                


  }

}



void setup() {
  Serial.begin(115200);
  connectWiFi();
  connectMQTT();
  client.setCallback(callback);
  client.subscribe("homeassistant/switch/servo");
  client.publish("homeassistant/switch/servo/availability", "online", true);
  servo1.attach(18);
  servo2.attach(15);
  pinMode(servoPin1, OUTPUT);
  pinMode(servoPin2, OUTPUT);
  //servo2.writeMicroseconds(1500); // 设置为中间位置（90度）
  pinMode(15, INPUT_PULLUP);  // 配置引脚2为输入，并启用上拉电阻
  attachInterrupt(digitalPinToInterrupt(15), stop, FALLING);  // 配置中断
  
}

void loop() {
  if (!client.connected()) {
    connectMQTT();
  }
  client.loop();
//  // 检查是否需要停止舵机
  if (lastActionTime > 0 && millis() - lastActionTime >= actionInterval) {
    servo1.detach();
    servo2.detach();
    lastActionTime = 0;
  }




}

<!---
ZongYuXian/ZongYuXian is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
