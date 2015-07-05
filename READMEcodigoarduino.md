# robotinho
codigo de arduino 

//Include the necessary libraries.
#include <SPI.h>
#include <WiFi.h>
#include <WiFiUdp.h>

int status = WL_IDLE_STATUS;
char ssid[] = "Open-UPCT";//"TP-LINK_E05002"; //  your network SSID (name) 
//char pass[] = "diego14567";    // your network password (use for WPA, or use as key for WEP)
int keyIndex = 0;            // your network key Index number (needed only for WEP)
//int ledPinrojo=3;
//int ledPinverde=2;
int izqA = 5; 
int izqB = 6; 
int derA = 9; 
int derB = 8; 
int ledpinrojo=10;
int ledpinverde=2;
int vel = 230; // Velocidad de los motores (0-255)

unsigned int localPort = 20;//2390;      // local port to listen on

char packetBuffer[255]; //buffer to hold incoming packet

WiFiUDP Udp;

void setup(){
  
  pinMode(derA, OUTPUT);
  pinMode(derB, OUTPUT);
  pinMode(izqA, OUTPUT);
  pinMode(izqB, OUTPUT);  
  pinMode(ledpinverde, OUTPUT);
  pinMode(ledpinrojo, OUTPUT);
   digitalWrite(ledpinrojo, HIGH);
       
  
  
  //IF YOU WANT TO TEST YOUR SHIELD AND SEE INFORMATION ON THE SERIAL MONTIOR,
  //UNCOMMENT THE FOLLOWING FUNCTION.
  //IF YOU WANT TO RUN THE MOTORS, COMMENT IT OUT AGAIN.
  Serial.begin(9600); 
    
  //UDP Configuration
  // check for the presence of the shield:
  if (WiFi.status() == WL_NO_SHIELD) {
    Serial.println("WiFi shield not present"); 
    // don't continue:
    while(true);
  } 
  
  // attempt to connect to Wifi network:
  while ( status != WL_CONNECTED) {
     
    Serial.print("Attempting to connect to SSID: ");
    Serial.println(ssid);
    // Connect to WPA/WPA2 network. Change this line if using open or WEP network:    
    status = WiFi.begin(ssid);

    // wait 10 seconds for connection:
    delay(10000);
  }
  Serial.println("Connected to wifi");
  //printWifiStatus();
  
  Serial.println("\nStarting connection to server...");
  // if you get a connection, report back via serial:
  Udp.begin(localPort);
  
  // print your WiFi shield's IP address:
  IPAddress ip = WiFi.localIP();
  Serial.print("IP Address: ");
  Serial.println(ip);
}

void loop() {
  // if there's data available, read a packet
  int packetSize = Udp.parsePacket();
  if(packetSize)
  {  digitalWrite(ledpinverde, HIGH);
    Serial.print("Received packet of size ");
    Serial.println(packetSize);
    Serial.print("From ");
    IPAddress remoteIp = Udp.remoteIP();
    Serial.print(remoteIp);
    Serial.print(", port ");
    Serial.println(Udp.remotePort());

    // read the packet into packetBufffer
    int len = Udp.read(packetBuffer,255);
    if (len >0) 
      packetBuffer[len]=0;
    Serial.println("Contents:");
    Serial.println(packetBuffer);
  }

  if(packetBuffer[0]=='s')
  {

    analogWrite(derA, vel);  // hacia atrás.
    analogWrite(izqA, vel); 
    analogWrite(derB, 0);  
    analogWrite(izqB, 0);
   
   
  
  }
  if(packetBuffer[0]=='w'){
    analogWrite(derB, vel);  // hacia delante.
    analogWrite(izqB, vel);
     analogWrite(derA, 0);  
    analogWrite(izqA, 0);
    }
  if(packetBuffer[0]=='q')
  {
      analogWrite(derB, 0);  // Detiene los Motores
  analogWrite(izqB, 0);
  analogWrite(derA, 0);
  analogWrite(izqA, 0);
  
  }
  if(packetBuffer[0]=='d'){ // hacia la derecha
    analogWrite(derB, 0);  
  analogWrite(izqB, vel-30);
  analogWrite(derA, vel-30);
  analogWrite(izqA, 0);
  }
   if(packetBuffer[0]=='a')//hacia izquierda
   {
  analogWrite(derB, vel-30);  
  analogWrite(izqB, 0);
  analogWrite(derA, 0);
  analogWrite(izqA, vel-30);
  }

  delay(100);
}

void printWifiStatus() {
  // print the SSID of the network you're attached to:
  Serial.print("SSID: ");
  Serial.println(WiFi.SSID());

  // print your WiFi shield's IP address:
  IPAddress ip = WiFi.localIP();
  Serial.print("IP Address: ");
  Serial.println(ip);

  // print the received signal strength:
  long rssi = WiFi.RSSI();
  Serial.print("signal strength (RSSI):");
  Serial.print(rssi);
  Serial.println(" dBm");
}
