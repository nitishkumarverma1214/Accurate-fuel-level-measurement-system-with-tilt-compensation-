# Accurate-fuel-level-measurement-system-with-tilt-compensation-
#include <ESP8266WiFi.h>
#include <PubSubClient.h>


const char* ssid = "k6power";
const char* password = "nitish1214";

#define ORG "28fhza"
#define DEVICE_TYPE "nodemuc"
#define DEVICE_ID "2799"
#define TOKEN "123456789"

# define echo1 D1
# define trig1 D2
# define echo2 D3
# define trig2 D4
# define echo3 D5
# define trig3 D6

char server[] = ORG ".messaging.internetofthings.ibmcloud.com";
char topic[] = "iot-2/evt/Data/fmt/json";
char authMethod[] = "use-token-auth";
char token[] = TOKEN;
char clientId[] = "d:" ORG ":" DEVICE_TYPE ":" DEVICE_ID;

WiFiClient wifiClient;
PubSubClient client(server, 1883, wifiClient);

void setup() {
 Serial.begin(115200);
pinMode(echo, INPUT);
pinMode(trig, OUTPUT);
 
 wifiConnect();
 
}
void wifiConnect() {
  Serial.print("Connecting to "); 
  Serial.print(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.print("nWiFi connected, IP address: ");
  Serial.println(WiFi.localIP());
}

void loop() {
 
int dist1 = calDistance(trig1, echo1);
int dist2 = calDistance(trig2, echo2);
int dist3 = calDistance(trig3, echo3);
int distance = dist1 + dist2 + dist3;
 PublishData(distance);
delay(1000);
 
 
}
int calDistance(int trig, int echo)
{
  digitalWrite(trig,LOW);
  delay(10);
  digitalWrite(trig,HIGH);
  delay(10);
  digitalWrite(trig,LOW);
  int duration = pulseIn(echo,HIGH);
 int distance = (duration * 0.034)/2; 
 return distance;
  }

void PublishData(int distance){
 if (!!!client.connected()) {
 Serial.print("Reconnecting client to ");
 Serial.println(server);
 while (!!!client.connect(clientId, authMethod, token)) {
 Serial.print(".");
 delay(500);
 }
 Serial.println();
 }
  String payload = "{\"d\":{\"distance\":";
  payload += distance;
  payload += "}}";
 Serial.print("Sending payload: ");
 Serial.println(payload);
  
 if (client.publish(topic, (char*) payload.c_str())) {
 Serial.println("Publish ok");
 } else {
 Serial.println("Publish failed");
 }
}
