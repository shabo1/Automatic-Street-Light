# Automatic-Street-Light

#include<SoftwareSerial.h> //include the software serial library

int sensor; //variable to store sensor values
int data; //variable to store mapped sensor values
int Light_status=10; //variable to store pump values
int threshold=55;

SoftwareSerial esp8266(3,4); //set the software serial pins RX=3,TX=4

#define SSID "Shabo"
#define PASS "sssssssss"

String sendAT(String command,const int timeout){
  String response="";
  esp8266.print(command);
  long int time = millis();
  while((time + timeout)> millis()){
    while(esp8266.available()){
      char c = esp8266.read();
      response += c;
    }
  }
  Serial.print(response);
  return response;
}

void connectwifi(){
  sendAT("AT\r\n",1000);
  sendAT("AT+CWMODE=1\r\n",1000); //call sendAT function to get esp8266 to station mode
  sendAT("AT+CWJAP=\""SSID"\",\""PASS"\"\r\n",2000); //AT command to connect with wifi network
  while(!esp8266.find("OK")){
    //wait for connection
  }
  sendAT("AT+CIFSR\r\n",1000); //AT command to print IP address on serial monitor
  sendAT("AT+CIPMUX=0\r\n",1000); //AT command to set esp8266 to single connection
}

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600); //begin the serial communication with baud rate 9600
  esp8266.begin(9600);//begin the software serial communication with baud rate 9600
  //WiFi.softAP(SSID,PASS);
  sendAT("AT+RST\r\n",2000); //call sendAT function to send reset AT command
  connectwifi();
  pinMode(8,OUTPUT);
}

void loop() {
  // put your main code here, to run repeatedly:
  sensor = analogRead(A0); //read raw sensor data and store it in variable sensor
  data = map(sensor,0,1023,0,100); //map the raw sensor values and store the result in data variable

  if(data<threshold){ //check if sensor value is less than threshold
    digitalWrite(8,HIGH); //switch on the light
    Light_status = 100; //update Light status variable value to 100
  }
else{
  digitalWrite(8,LOW); //switch off the light
  Light_status=10; //update the light_status variable value to 10
}
String sensor_value = String(data); //convert integer to string datatype
Serial.print("Light intensity:");
Serial.println(data); //print light intensity on serial monitor

String Light_status1 = String(Light_status); //convert integer to string datatype
Serial.print("Light status:");
Serial.println(Light_status); //print light status on serial monitor

String threshold1 = String(threshold); //convert integer to string data type
Serial.print("Threshold:");
Serial.println(threshold); //print threshold on serial monitor

updateTS(sensor_value,Light_status1,threshold1); //call the function to update ThingSpeak channel
delay(1000);
}

void updateTS(String S,String L,String T){
  Serial.println("");
  sendAT("AT+CIPSTART=\"TCP\",\"api.thingspeak.com\",80\r\n",1000);
  delay(2000);
  String cmdlen;
  String cmd = "GET /update?key=F2WE2JLE0NNTUUFR&field1="+L+"&field2="+S+"&field3="+T+"\r\n"; //update the Light intensity, Light status and threshold values to Thingspeak channel
  cmdlen= cmd.length();
  sendAT("AT+CIPSEND="+cmdlen+"\r\n",2000);
  esp8266.print(cmd);
  Serial.print("");
  sendAT("AT+CIPCLOSE\r\n",2000);
  Serial.println("");
  
}
