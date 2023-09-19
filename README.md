- 👋 Hi, I’m @ZongYuXian
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...


#include <WiFi.h>
#include <PubSubClient.h>
#include <ESP32Servo.h>
// WiFi连接信息
const char* ssid = "TP-LINK_Zong";
const char* password = "zongyuxian";
// MQTT服务器信息
const char* mqtt_server = "192.168.0.102";
const int mqtt_port = 1883;
const char* mqtt_username = "admin";
const char* mqtt_password = "zyxblr3137";

// 创建一个Servo对象
Servo servo1;
Servo servo2;
// 舵机引脚
const int servoPin1 = 25;
const int servoPin2 = 5;
int servo1Angle = 0;
int servo2Angle = 0;



bool servo1Reverse = false;
bool servo2Reverse = false;
bool servo1Stopped = true;
bool servo2Stopped = true;
bool isServo1Attached = false;
bool isServo1Rotating = true;


int servoPosition = 0;  // 初始化舵机位置
bool stopRotation = false;  // 停止旋转标志位

int servo1Rotations = 0;




//########################################################################################################################################################################################
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


// 接收MQTT消息的回调函数
void callback(char* topic, byte* payload, unsigned int length) {
  // 将接收到的消息转换为字符串
  String message = "";
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  // 发布舵机的状态
  if (message == "0" || message == "1" || message == "2" || message == "3" || message == "4") {
    client.publish("servo/state", message.c_str());
  }
  // 发布舵机的可用性
  client.publish("homeassistant/switch/servo/availability", "online");
  }

   // 根据接收到的消息控制舵机
  if (message == "0") 
  {
    // 输入0时，舵机1持续正转
    if (!isServo1Rotating) {
      servo1.attach(25);
      servo1.write(360);
      isServo1Rotating = true;
    } else {
      servo1.detach();
      isServo1Rotating = false;
    }
              
  } else if (message == "1") {
    
      
      servo1.attach(25);  // 连接舵机到引脚2
      servo1.writeMicroseconds(2500);
      servo1.write(0);  // 旋转舵机到0度
      delay(100);  // 延迟1秒，让舵机旋转到指定角度
      servo1.detach();  // 断开舵机与引脚的连接，停止舵机
     


  }

  
// 根据接收到的消息控制舵机2
   else if (message == "2") {

      
      servo2.writeMicroseconds(2000); // 将舵机2的脉冲宽度设置为2000微秒（20毫秒）
      servo2.write(180);
      delay(1000);
      
      servo2.writeMicroseconds(1500); // 将舵机2的脉冲宽度设置为1500微秒（1.5毫秒），用于停止
      servo2.write(180);
      
      

      
        
        
          
  } else if (message == "3") {
           

      servo2.writeMicroseconds(1000); // 将舵机2的脉冲宽度设置为2000微秒（20毫秒）
      servo2.write(90);
      delay(1000);
      
      servo2.writeMicroseconds(1500); // 将舵机2的脉冲宽度设置为1500微秒（1.5毫秒），用于停止
      servo2.write(90);
      
      
 

      
  }  else if (message == "4") {
    


         // 输入0时，舵机1持续正转
        if (!isServo1Rotating) {
          servo1.attach(25);
          servo1.write(0);
          isServo1Rotating = true;
        } else {
          servo1.detach();
          isServo1Rotating = false;
        }



  }  else if (message == "5") {
    

      servo2.writeMicroseconds(1000); // 将舵机2的脉冲宽度设置为2000微秒（20毫秒） 
      servo2.write(0);
      delay(1000);
      servo2.writeMicroseconds(1500); // 将舵机2的脉冲宽度设置为1500微秒（1.5毫秒），用于停止
      servo2.write(0);
      
      
    }



    else if (message == "6") {
    

          // 输入0时，舵机1持续正转
        if (!isServo1Rotating) {
          servo1.attach(25);
          servo1.write(360);
          isServo1Rotating = true;
        } else {
          servo1.detach();
          isServo1Rotating = false;
        }
      
      
    }

  }








void setup() {
  Serial.begin(115200);
  connectWiFi();
  connectMQTT();
  client.setCallback(callback);
  client.subscribe("homeassistant/switch/servo");
  client.publish("homeassistant/switch/servo/availability", "online", true);

  servo1.attach(25);
  servo2.attach(5);

  // 初始化位置
  //servo1.write(0);
  // 将舵机1初始化为停止状态
  servo1.write(90);
  servo2.write(0);
  servo2.setPeriodHertz(50); // 设置舵机2的频率为50Hz，对应时基脉冲为20毫秒
  
  // 设置初始状态为停止
  servo1Stopped = true;
  servo2Stopped = true;



  
}

void loop() {
  if (!client.connected()) {
    connectMQTT();
  }
  client.loop();



}


<!---
ZongYuXian/ZongYuXian is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
