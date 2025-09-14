#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <ESP8266WebServer.h>

// Define the SSID and Password for the ESP8266's hosted network
const char* ssid = "CarControl";
const char* password = "12345678";

ESP8266WebServer server(80);

// Define the GPIO pins for the motor driver
const byte L298N_A_pin = D1;
const byte L298N_A_In1_pin = D2;
const byte L298N_A_In2_pin = D3;
const byte L298N_B_In3_pin = D4;
const byte L298N_B_In4_pin = D5;
const byte L298N_B_pin = D6;

// Function to control motor speed and direction
void motorSpeed(int prmA, byte prmA1, byte prmA2, int prmB, byte prmB1, byte prmB2) {
  analogWrite(L298N_A_pin, prmA);
  analogWrite(L298N_B_pin, prmB);

  digitalWrite(L298N_A_In1_pin, prmA1);
  digitalWrite(L298N_A_In2_pin, prmA2);
  digitalWrite(L298N_B_In3_pin, prmB1);
  digitalWrite(L298N_B_In4_pin, prmB2);
}

// Handle the root URL by sending a simple message
void handleRoot() {
  server.send(200, "text/plain", "Hello from ESP8266 AP!");
}

// Handle /car URL for controlling the car via web interface
void handleCar() {
  String message = "";
  int BtnValue = 0;
  for (uint8_t i = 0; i < server.args(); i++) {
    if (server.argName(i) == "a") {
      BtnValue = server.arg(i).toInt();
    }
  }

  switch (BtnValue) {
    case 1: // Left turn
      motorSpeed(900, LOW, LOW, 1023, HIGH, LOW); 
      break;
    case 2: // Straight forward
      motorSpeed(1023, HIGH, LOW, 1023, HIGH, LOW); 
      break;
    case 3: // Right turn
      motorSpeed(1023, HIGH, LOW, 900, LOW, LOW); 
      break;
    case 4: // Full left turn
      motorSpeed(900, LOW, HIGH, 900, HIGH, LOW); 
      break;
    case 5: // Stop
      motorSpeed(0, LOW, LOW, 0, LOW, LOW); 
      break;
    case 6: // Full right turn
      motorSpeed(900, HIGH, LOW, 900, LOW, HIGH); 
      break;
    case 7: // Reverse left
      motorSpeed(900, LOW, LOW, 1023, LOW, HIGH); 
      break;
    case 8: // Full reverse
      motorSpeed(900, LOW, HIGH, 900, LOW, HIGH);      
      break;
    case 9: // Reverse right
      motorSpeed(1023, LOW, HIGH, 900, LOW, LOW); 
      break;
    default:
      break;
  }

  // Build the HTML web page to control the car
  message += "<html><head><title>Car Control</title></head><body>";
  message += "<h3>IoT Based Robotic Car Control Web Server</h3>";
  message += "<table>";
  message += "<tr><td><a href=\"/car?a=1\"><button style=\"width:100;height:100;font-size:20px;\">Left</button></a></td>";
  message += "<td><a href=\"/car?a=2\"><button style=\"width:100;height:100;font-size:20px;\">Forward</button></a></td>";
  message += "<td><a href=\"/car?a=3\"><button style=\"width:100;height:100;font-size:20px;\">Right</button></a></td></tr>";
  message += "<tr><td><a href=\"/car?a=4\"><button style=\"width:100;height:100;font-size:20px;\">Full Left</button></a></td>";
  message += "<td><a href=\"/car?a=5\"><button style=\"width:100;height:100;font-size:20px;\">Stop</button></a></td>";
  message += "<td><a href=\"/car?a=6\"><button style=\"width:100;height:100;font-size:20px;\">Full Right</button></a></td></tr>";
  message += "<tr><td><a href=\"/car?a=7\"><button style=\"width:100;height:100;font-size:20px;\">Reverse Left</button></a></td>";
  message += "<td><a href=\"/car?a=8\"><button style=\"width:100;height:100;font-size:20px;\">Reverse</button></a></td>";
  message += "<td><a href=\"/car?a=9\"><button style=\"width:100;height:100;font-size:20px;\">Reverse Right</button></a></td></tr>";
  message += "</table></body></html>";

  server.send(200, "text/html", message);
}

// Handle not found URLs
void handleNotFound() {
  String message = "File Not Found\n\n";
  message += "URI: ";
  message += server.uri();
  message += "\nMethod: ";
  message += (server.method() == HTTP_GET ? "GET" : "POST");
  message += "\nArguments: ";
  message += server.args();
  message += "\n";
  for (uint8_t i = 0; i < server.args(); i++) {
    message += " " + server.argName(i) + ": " + server.arg(i) + "\n";
  }
  server.send(404, "text/plain", message);
}


void setup() {
  // Set motor control pins as output
  pinMode(L298N_A_In1_pin, OUTPUT);
  pinMode(L298N_A_In2_pin, OUTPUT);
  pinMode(L298N_B_In3_pin, OUTPUT);
  pinMode(L298N_B_In4_pin, OUTPUT);

  // Initialize all motor control pins to LOW
  digitalWrite(L298N_A_In1_pin, LOW);
  digitalWrite(L298N_A_In2_pin, LOW);
  digitalWrite(L298N_B_In3_pin, LOW);
  digitalWrite(L298N_B_In4_pin, LOW);

  Serial.begin(115200);

  // Configure the ESP8266 as an Access Point
  WiFi.mode(WIFI_AP);
  WiFi.softAP(ssid, password);

  Serial.print("Access Point \"");
  Serial.print(ssid);
  Serial.println("\" started");
  Serial.print("IP address: ");
  Serial.println(WiFi.softAPIP());

  // Define server routes
  server.on("/", handleRoot);
  server.on("/car", handleCar);
  server.onNotFound(handleNotFound);

  server.begin();
  Serial.println("HTTP server started");
}

void loop() {
  server.handleClient();
}
