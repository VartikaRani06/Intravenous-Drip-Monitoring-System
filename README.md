# Intravenous-Drip-Monitoring-System
This an arduino  based system that allows the monitor the level of the drip with the help of an iot app.


CODE:
#include <BlynkSimpleEsp8266.h>
#include <NewPing.h>

// Define the pins for the ultrasonic sensor
#define TRIGGER_PIN D1
#define ECHO_PIN D2

// Define the pin for the buzzer
#define BUZZER_PIN D3

// Define the pin for the pulse sensor
#define PULSE_SENSOR_PIN A0

// Define the maximum distance in centimeters
#define MAX_DISTANCE 100

// Create an instance of the NewPing library
NewPing sonar(TRIGGER_PIN, ECHO_PIN, MAX_DISTANCE);

// Blynk authentication token
char auth[] = "rfUJjBW3jg2DJNE95E5mgRQEILk3uGyZ";

// Buzzer state variable
int buzzerState = LOW;

void setup() {
  // Initialize the serial communication
  Serial.begin(9600);

  // Connect to the Blynk server
  Blynk.begin(auth, "realme 7", "yashaswi");

  // Set the buzzer pin and pulse sensor pin as outputs
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(PULSE_SENSOR_PIN, INPUT);

  // Set the initial state of the buzzer
  digitalWrite(BUZZER_PIN, buzzerState);
}

void loop() {
  // Process Blynk communication
  Blynk.run();

  // Read the analog value from the Pulse Sensor
  int sensorValue = analogRead(PULSE_SENSOR_PIN);

  // Convert the analog value to a voltage between 0 and 3.3V
  float voltage = sensorValue * (3.3 / 1023.0);

  // Calculate the heart rate based on the sensor voltage
  int heartRate = map(voltage, 0.6, 2.6, 40, 220)+44;

  // Print the heart rate
 if(heartRate>60&& heartRate<120 )
  {
  Serial.print("Heart Rate: ");
  Serial.print(heartRate);
  Serial.println(" bpm");
  }

  // Send a ping to the ultrasonic sensor and get the distance in centimeters
  int distance = sonar.ping_cm();

  // Check if the distance is within the desired range
  if (distance > 7 && distance <= 30) {
    // Turn on the buzzer if it's not already on
    if (buzzerState == LOW) {
      digitalWrite(BUZZER_PIN, HIGH);
      buzzerState = HIGH;
      Serial.println("Object detected!");
      Blynk.virtualWrite(V2, 1);
    }
  } else {
    // Turn off the buzzer if it's not already off
    if (buzzerState == HIGH) {
      digitalWrite(BUZZER_PIN, LOW);
      buzzerState = LOW;
      Serial.println("Object not detected!");
      Blynk.virtualWrite(V2, 0);
    }
  }

  // Display the distance on the serial monitor
  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");
 
  // Update Blynk virtual pins
 
  Blynk.virtualWrite(V3, heartRate);
  Blynk.virtualWrite(V1, distance);
  

  // Wait for a short delay before taking the next measurement
  delay(1000);
}

