Vejledning: Wi-Fi Styret Bil med ESP-01 og Arduino
Hvordan man opsætter en ESP-01 som et Access Point, der modtager UDP-data fra en joystick-app og sender det videre til en Arduino via Serial-kommunikation. JOYstick appen der er kode til her er ”UDP Joystick” fra google play.
1. Hardware Setup (Vigtigt!)
ESP-01 kører på 3.3V. Forbindes den til 5V, går den i stykker.
•	ESP-01 VCC: Forbindes til 3.3V.
•	ESP-01 GND: Forbindes til Arduino GND (Fælles stel).
•	ESP-01 CH_PD (EN): Skal forbindes til 3.3V for at aktivere chippen.// er det nødvendigt
•	ESP-01 TX: Forbindes til Arduino Pin 2.
•	ESP-01 RX: Forbindes til Arduino Pin 3 (Brug en spændingsdeler, da Arduino sender 5V!).
2. Kode til ESP-01 (UDP Bro)
Denne kode gør ESP-01 til et trådløst netværk. Den lytter efter UDP-pakker på port 8888 og sender dem ud af sin TX-pin.
Indstillinger i Arduino IDE:
Board: Generic ESP8266 Module
Husk at sætte ESP-01 i programmerings mode ved at sætte GPIO0 til gnd ved opstart
Kode
#include <ESP8266WiFi.h>
#include <WiFiUDP.h>

const char* ssid = "Bil_Team_1"; // Skift navnet så I kan kende jeres bil
const char* password = "elektroteknik";

WiFiUDP udp;
unsigned int localPort = 8888;

void setup() {
  Serial.begin(115200);
  WiFi.softAP(ssid, password);
  udp.begin(localPort);
}

void loop() {
  int packetSize = udp.parsePacket();
  if (packetSize) {
    char packetBuffer[255];
    int len = udp.read(packetBuffer, 255);
    if (len > 0) {
      packetBuffer[len] = 0;
    }
    Serial.println(packetBuffer); // Sender strengen til Arduinoen
  }
}


3. Kode til Arduino 
Arduinoen modtager den lange talstreng (f.eks. 1500150015001500) og skal dele den op, så vi kan bruge tallene til at styre motorerne.
#include <SoftwareSerial.h>
SoftwareSerial espSerial(2, 3); // RX på pin 2 (fra ESP TX), TX på pin 3 i kan også bruge ex. 10 og 11
void setup() {
  Serial.begin(9600);      // Til computeren
  espSerial.begin(115200); // Til ESP-01
  Serial.println("Systemet er klar...");
}

void loop() {
  if (espSerial.available()) {
    String rawData = espSerial.readStringUntil('\n');
    rawData.trim();

    // Vi forventer 16 tegn (4 kanaler af 4 cifre)
    if (rawData.length() >= 16) {
      // OPGAVE: Split strengen op i 4 variabler
      int ch1 = rawData.substring(0, 4).toInt();
      int ch2 = rawData.substring(4, 8).toInt();
      int ch3 = rawData.substring(8, 12).toInt();
      int ch4 = rawData.substring(12, 16).toInt();

      // Se tallene i Serial Monitor
      Serial.print("Kanal 2 (Frem/Bak): "); Serial.println(ch2);

      // OPGAVE: Her skal I programmere jeres mapping og motorstyring
      // Tip: Hvis ch2 > 1550, skal bilen køre fremad.
    }
  }
}

1.	Forbindelse: Få hul igennem så I kan se joystick-tallene ændre sig i Arduinoens Serial Monitor.
2.	Mapping: Brug funktionen map() til at omdanne værdierne fra joystick-skalaen (1000-2000) til motor-skalaen (0-255).
3.	Dødbånd: Indbyg et lille "dødbånd" omkring 1500 (f.eks. 1480 til 1520), hvor motoren står helt stille, så bilen ikke snurrer, når joysticket er i midten.
4.	Logik: Skriv koden der aktiverer jeres motor-driver (f.eks. L298N) baseret på de værdier I har modtaget.

