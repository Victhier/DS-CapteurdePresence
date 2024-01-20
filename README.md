#include <Servo.h>
#include <Ultrasonic.h>

// Déclaration des broches
const int feuRouge = 3;
const int feuVert = 4;
const int moteur = 9;
const int echo = 2;  // Broche de l'écho du capteur ultrasonique
const int trig = 5;  // Broche du déclencheur du capteur ultrasonique

// Initialisation du servo
Servo monServo;

// Initialisation du capteur ultrasonique
Ultrasonic ultrason(trig, echo);

// Variables pour le suivi du temps de détection
unsigned long tempsDebutDetection = 0;
bool detectionEnCours = false;

void setup() {
  // Initialisation des broches en sortie
  pinMode(feuRouge, OUTPUT);
  pinMode(feuVert, OUTPUT);

  // Attache le servo à la broche
  monServo.attach(moteur);

  // Initialise la communication série
  Serial.begin(9600);
}

void loop() {

  // Allume la LED rouge
  digitalWrite(feuRouge, HIGH);

  // Mesure la distance avec le capteur ultrasonique
  float distance = ultrason.read();

  // Si la distance est inférieure à 10 cm, considère cela comme une détection
  if (distance < 10) {
    if (!detectionEnCours) {
      tempsDebutDetection = millis();
      detectionEnCours = true;
    }

    // Rotation du servo à 90 degrés après 3 secondes de détection
    if (millis() - tempsDebutDetection > 3000) {
      monServo.write(90);
      delay(500);

      // Éteint la LED rouge et allume la LED verte pendant 5 secondes
      digitalWrite(feuRouge, LOW);
      digitalWrite(feuVert, HIGH);
      delay(5000);

      // Éteint la LED verte
      digitalWrite(feuVert, LOW);
      digitalWrite(feuRouge, HIGH);

      // Rotation du servo pour revenir à la position initiale (plus lentement)
    for (int angle = 90; angle >= 0; angle--) {
      monServo.write(angle);
      delay(50); // Augmentez ce délai pour ralentir la rotation du servo
    }
      // Réinitialise le suivi de la détection
      detectionEnCours = false;
    }
  } else {
    // Si aucune détection, feuRouge allumé et réinitialise le servo
    digitalWrite(feuRouge, HIGH);
    digitalWrite(feuVert, LOW);
    monServo.write(0);
    detectionEnCours = false;
  }
}
