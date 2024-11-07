// Define Pins
#define trigPin 11         // Trigger pin for ultrasonic sensor
#define echoPin 10         // Echo pin for ultrasonic sensor
#define greenLED 2         // Green LED for safe distance
#define yellowLED 3        // Yellow LED for caution distance
#define redLED 5           // Red LED for close distance

// Fixed Thresholds
const int upperThreshold = 20;   // Fixed upper threshold in cm
const int lowerThreshold = 10;   // Fixed lower threshold in cm

void setup() {
  // Initialize LED pins as outputs
  pinMode(greenLED, OUTPUT);
  pinMode(yellowLED, OUTPUT);
  pinMode(redLED, OUTPUT);

  // Initialize ultrasonic sensor pins
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  // Initialize serial communication for debugging
  Serial.begin(9600);

  // Initial test cases to verify LED functionality
  Serial.println("Running initial test cases...");
  testLEDStates(25); // Test for safe distance (Green LED)
  delay(1000);
  testLEDStates(15); // Test for caution distance (Yellow LED)
  delay(1000);
  testLEDStates(5);  // Test for close distance (Red LED)
  delay(1000);
  Serial.println("Initial tests completed. Starting normal operation.");
}

void testLEDStates(long distance) {
  // Test LED states based on a given distance
  Serial.print("Testing distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  if (distance > upperThreshold) {
    digitalWrite(greenLED, HIGH);    // Green LED on (safe distance)
    digitalWrite(yellowLED, LOW);
    digitalWrite(redLED, LOW);
  } else if (distance > lowerThreshold) {
    digitalWrite(greenLED, LOW);
    digitalWrite(yellowLED, HIGH);   // Yellow LED on (caution distance)
    digitalWrite(redLED, LOW);
  } else {
    digitalWrite(greenLED, LOW);
    digitalWrite(yellowLED, LOW);
    digitalWrite(redLED, HIGH);      // Red LED on (close distance)
  }

  // Wait a moment to observe the LED state
  delay(1000);

  // Turn off all LEDs after the test
  digitalWrite(greenLED, LOW);
  digitalWrite(yellowLED, LOW);
  digitalWrite(redLED, LOW);
}

long measureDistance() {
  // Trigger the ultrasonic sensor to send a pulse
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  // Read the echo pin
  long duration = pulseIn(echoPin, HIGH);

  // Check if no echo was received (timeout)
  if (duration == 0) {
    Serial.println("No echo received (timeout). Object might be out of range.");
    return -1;  // Indicates no valid reading
  }

  long distance = duration * 0.034 / 2;

  // Debug: Print the raw duration and calculated distance
  Serial.print("Duration: ");
  Serial.print(duration);
  Serial.print(" | Distance: ");
  Serial.println(distance);

  return distance;
}

void loop() {
  // Measure distance using the ultrasonic sensor
  long distance = measureDistance();

  // Check if we got a valid distance measurement
  if (distance == -1) {
    Serial.println("Object out of range. Waiting...");
    digitalWrite(greenLED, LOW);
    digitalWrite(yellowLED, LOW);
    digitalWrite(redLED, LOW);
    delay(500); // Wait before measuring again
    return;  // Skip to the next loop iteration
  }

  // LED control logic based on fixed thresholds
  if (distance > upperThreshold) {
    digitalWrite(greenLED, HIGH);    // Green LED on (safe distance)
    digitalWrite(yellowLED, LOW);
    digitalWrite(redLED, LOW);
  } else if (distance > lowerThreshold) {
    digitalWrite(greenLED, LOW);
    digitalWrite(yellowLED, HIGH);   // Yellow LED on (caution distance)
    digitalWrite(redLED, LOW);
  } else {
    digitalWrite(greenLED, LOW);
    digitalWrite(yellowLED, LOW);
    digitalWrite(redLED, HIGH);      // Red LED on (close distance)
  }

  // Delay to prevent rapid changes and allow for stable readings
  delay(500);
}
