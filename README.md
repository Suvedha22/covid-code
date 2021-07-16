# covid-code
 Program code for generic ESP8266
// ----------(c) 
Electronics-project-hub-------- //
#include "ThingSpeak.h“
#include <ESP8266WiFi.h>
//------- WI-FI details ----------//
char ssid[] = "XXXXXXXXXXX"; // SSID here
char pass[] = "YYYYYYYYYYY"; // Passowrd here
//--------------------------------////----------- Channel details ----------------//
unsigned long Channel_ID = 123456; // Channel ID
const char * myWriteAPIKey = "ABCDEFG1234"; //Your write API key
//-------------------------------------------//
const int Field_Number_1 = 1;
const int Field_Number_2 = 2;
String value = "";
int value_1 = 0, value_2 = 0;
int x, y;
WiFiClient client;
 void setup()
{
Serial.begin(115200);
WiFi.mode(WIFI_STA);
ThingSpeak.begin(client);
internet();
}
void loop()
{
internet();
if (Serial.available() > 0)
{
delay(100);
while (Serial.available() > 0)
{
value = Serial.readString();
if (value[0] == '*')
{
if (value[5] == '#')
{
value_1 = ((value[1] - 0x30) * 10 + (value[2] - 0x30));
value_2 = ((value[3] - 0x30) * 10 + (value[4] - 0x30));
[16/07, 12:03 pm] suruthi: else if (value[6] == '#')
{
value_1 = ((value[1] - 0x30) * 100 + (value[2] - 0x30) * 10 + (value[3] - 0x30)); 
value_2 = ((value[4] - 0x30) * 10 + (value[5] - 0x30)); 
}
}
}
}
upload();
}
void internet()
{
if (WiFi.status() != WL_CONNECTED)
{
while (WiFi.status() != WL_CONNECTED)
{
WiFi.begin(ssid, pass);
delay(5000);
}
}
 }
void upload()
{
ThingSpeak.writeField(Channel_ID, Field_Number_1, value_1, myWriteAPIKey);
delay(15000);
ThingSpeak.writeField(Channel_ID, Field_Number_2, value_2, myWriteAPIKey);
delay(15000);
value = "";
}
// ----------(c) Electronics-project-hub-------- //
Program code for Adrenio
#include <SoftwareSerial.h>
#include <OneWire.h>
#include <DallasTemperature.h>
#define USE_ARDUINO_INTERRUPTS true
#include <PulseSensorPlayground.h>
SoftwareSerial esp(10, 11);
#define ONE_WIRE_BUS 9
#define TEMPERATURE_PRECISION 12
OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);
DeviceAddress tempDeviceAddress;
int numberOfDevices, temp, buzzer = 8;
const int PulseWire = A0;
int myBPM, Threshold = 550;
PulseSensorPlayground pulseSensor;unsigned long previousMillis = 0;
const long interval = 5000;
void setup()
{
Serial.begin(9600);
esp.begin(115200);
sensors.begin();
numberOfDevices = sensors.getDeviceCount();
pulseSensor.analogInput(PulseWire);
pulseSensor.setThreshold(Threshold);
pulseSensor.begin();
pinMode(buzzer, OUTPUT);
digitalWrite(buzzer, HIGH);
delay(1500);
digitalWrite(buzzer, LOW);
}
void loop()
{
myBPM = pulseSensor.getBeatsPerMinute();
if (pulseSensor.sawStartOfBeat())
{
beep();
Serial.println(myBPM);
delay(20);
}
sensors.requestTemperatures();
for (int i = 0; i < numberOfDevices; i++)
{
if (sensors.getAddress(tempDeviceAddress, i))
{
temp = printTemperature(tempDeviceAddress);
Serial.println(temp);
}
}
upload();

int printTemperature(DeviceAddress deviceAddress){
int tempC = sensors.getTempC(deviceAddress);
return tempC;
}
void beep(){
digitalWrite(buzzer, HIGH);
delay(150);
digitalWrite(buzzer, LOW);
}
void upload(){
unsigned long currentMillis = millis();
if (currentMillis - previousMillis >= interval){
previousMillis = currentMillis;
esp.print('*');
esp.print(myBPM); 
esp.print(temp);
esp.println('#');
}
}
// ----------(c) Electronics-project-hub -------------//
