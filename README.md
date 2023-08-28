# smartparking
#include <SPI.h>
#include <Ethernet.h>
#define echoPin 4  // attach pin D2 Arduino to pin Echo of HC-SR04
#define trigPin 5 //attach pin D3 Arduino to pin Trig of HC-SR04
int red_light_pin= 11;
int green_light_pin = 10;
int blue_light_pin = 9;

// defines variables
long duration; // variable for the duration of sound wave travel
int distance; // variable for the distance measurement
byte mac[] = { 0x90, 0xA2, 0xDA,0x00,0xF9, 0x5D };
// Ip, gateway i subnet prepisati sa postojeceg racunala ciji kabel uzimamo
byte ip[] = { 161,53,168, 104}; // IP adresa
byte gateway[] = { 161,53,168, 1}; // Adresa Gateway ureï¿½aja
byte subnet[] = { 255,255,255,128}; // Subnet maska

EthernetServer server(80); // Objekt EthernetServer na portu 80, http
void setup() {
  // pocetak ethernet veze
Ethernet.begin(mac, ip,gateway,subnet);
// startanje servera
server.begin();
// startanje serijske komunikacije

  pinMode(red_light_pin, OUTPUT);
  pinMode(green_light_pin, OUTPUT);
  pinMode(blue_light_pin, OUTPUT);
  
  
  pinMode(trigPin, OUTPUT); // Sets the trigPin as an OUTPUT
  pinMode(echoPin, INPUT); // Sets the echoPin as an INPUT
  Serial.begin(9600); // // Serial Communication is starting with 9600 of baudrate speed
 
}
void loop() {
  // Clears the trigPin condition
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  // Sets the trigPin HIGH (ACTIVE) for 10 microseconds
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  // Reads the echoPin, returns the sound wave travel time in microseconds
  duration = pulseIn(echoPin, HIGH);
  // Calculating the distance
  distance = duration * 0.034 / 2; // Speed of sound wave divided by 2 (go and back)
  // Displays the distance on the Serial Monitor
  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  // prisluskivanje klijenata
EthernetClient client = server.available();
if (client) { // ako postoji klijent
boolean currentLineIsBlank = true; //
while (client.connected()) { // dok je klijent spojen
if (client.available()) { // i postoji komunijacina sa klijentom
char c = client.read(); // ocitaj znak zahtjeva sa strane klijenta
if (c == '\n' && currentLineIsBlank) {
// ako je priljeni znak prazna linija i currentLineIsBlank=true
// Generiranje HTML stranice
// salje se HTTP header
client.println("HTTP/1.1 200 OK");
client.println("Content-Type: text/html");
client.println();
// tijelo html stranice

client.println("Trenutno stanje parkirnog mjesta: ");

if (digitalRead(2)==1 && digitalRead(3)==0){ // ako s D2 očita 1 (crvena boja) i s D3 0 (zelena boja), na web stranicu ispisuje zauzeto, a u drugim slučajevima slobodno
  client.print("ZAUZETO");
  }
  else {
     client.print("SLOBODNO");
    }
break;
}
if (c == '\n') {
// ako je primljena prazna linija, tj znak za novu liniju
// pocetak nove linije
currentLineIsBlank = true;
}
else if (c != '\r') { // ako je primljen znak razlicit od return
// primljen je znak na sadasnoj liniji
currentLineIsBlank = false;
}
}
}
// pauziranje de preglednik primi podatke
delay(10);
// zatvaranje veze
client.stop();
delay(100);
}
if(distance < 15){ // ako je udaljenost koju mjeri UZV senzor manja od 15 cm ...
   RGB_color(255,0,0); // RGB LED svjetli crveno
   digitalWrite(2,1); // na D2 šalje 1
   digitalWrite(3,0); // na D3 šalje 0
    }
    else{
      RGB_color(0,255,0); // RGB LED svjetli zeleno
       digitalWrite(2,0); // na D2 šalje 0
       digitalWrite(3,1); // na D3 šalje 1
      }
}

void RGB_color(int red_light_value, int green_light_value,int blue_light_value) // funkcija koju zovemo za postavljanje boje RGB LED-ice
 {
  analogWrite(red_light_pin, red_light_value);
  analogWrite(green_light_pin, green_light_value);
   analogWrite(blue_light_pin, blue_light_value);
}
