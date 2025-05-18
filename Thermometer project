// Include biblioteca pentru ecranul LCD cu interfață I2C
#include <LiquidCrystal_I2C.h>
// Include biblioteca pentru comunicația serială pe alți pini (SoftwareSerial)
#include <SoftwareSerial.h>

// Inițializează obiectul LCD cu adresa I2C 0x27 și dimensiune 16x2 caractere
LiquidCrystal_I2C lcd(0x27, 16, 2);     

// Creează un port serial software pe pinii D2 (Rx) și D3 (Tx) pentru Bluetooth
SoftwareSerial bt(2, 3);                 

// Definirea pinilor pentru senzorul de temperatură și pentru componentele de ieșire
const int tempPin = A0;                  // Pinul analogic conectat la senzorul de temperatură (ex: LM35)
const int ledBluePin = 6;                // LED pentru temperatură scăzută
const int ledYellowPin = 5;              // LED pentru temperatură medie
const int ledRedPin = 4;                 // LED pentru temperatură ridicată
const int buzzerPin = 7;                 // Pin pentru buzzer

// Variabile de configurare
bool useFahrenheit = false;              // False = °C, True = °F
String input = "";                       // Variabilă pentru citirea comenzilor Bluetooth
float alertThreshold = 30.0;             // Prag pentru declanșarea alertei (în °C)
bool alertShown = false;                 // Evită repetarea alertei

// Funcția de inițializare - rulează o singură dată la pornirea programului
void setup() {
  bt.begin(9600);                        // Inițializează comunicația Bluetooth la 9600 bps
  Serial.begin(9600);                    // Inițializează portul serial USB pentru debugging
  
  lcd.init();                            // Inițializează ecranul LCD
  lcd.backlight();                       // Activează iluminarea LCD-ului
  lcd.print("Initializing...");          // Afișează mesaj de pornire
  delay(1000);                           // Așteaptă 1 secundă
  lcd.clear();                           // Curăță ecranul LCD

  // Setează LED-urile și buzzer-ul ca ieșiri
  pinMode(ledBluePin, OUTPUT);
  pinMode(ledYellowPin, OUTPUT);
  pinMode(ledRedPin, OUTPUT);
  pinMode(buzzerPin, OUTPUT);
  
  // Se asigură că toate LED-urile și buzzer-ul sunt oprite la start
  digitalWrite(ledBluePin, LOW);
  digitalWrite(ledYellowPin, LOW);
  digitalWrite(ledRedPin, LOW);
  digitalWrite(buzzerPin, LOW);
}

// Funcția principală - rulează continuu după setup()
void loop() {

  // Citirea comenzilor din modulul Bluetooth
  while (bt.available()) {
    char c = bt.read();          // Citește caracterul de la Bluetooth
    if (c == '\n') {             // Dacă se primește ENTER (\n), se consideră comandă completă
      processCommand(input);     // Procesează comanda
      input = "";                // Golește bufferul pentru următoarea comandă
    } else {
      input += c;                // Adaugă caracterul la buffer
    }
  }

  // Citește tensiunea de la senzorul LM35 și o convertește în grade Celsius
  float voltage = analogRead(tempPin) * 5.0 / 1023.0; // Conversie tensiune (0-5V)
  float tempC = voltage * 100.0;                      // 10mV = 1°C, deci 1V = 100°C
  float tempDisplay = tempC;                          // Temperatură de afișat (poate fi convertită în Fahrenheit)
  String unit = "C";                                  // Unitatea de temperatură

  // Dacă e setat Fahrenheit, se face conversia
  if (useFahrenheit) {
    tempDisplay = tempC * 9.0 / 5.0 + 32.0;
    unit = "F";
  }

  // Verifică dacă temperatura a depășit pragul și declanșează alertă
  if (tempC > alertThreshold && !alertShown) {
    showAlert(tempC);  
    alertShown = true; 
  } 
  // Dacă temperatura revine la normal, oprește alerta
  else if (tempC <= alertThreshold && alertShown) {
    removeAlert();    
    alertShown = false; 
  }

  // Afișează temperatura pe LCD dacă e activ
  lcd.setCursor(0, 0);               // Setează cursorul la începutul primei linii
  lcd.print("Temp: ");
  lcd.print(tempDisplay, 1);         // Afișează temperatura cu 1 zecimală
  lcd.print(" ");
  lcd.print(unit);
  lcd.print("     ");                // Spații pentru a șterge caractere vechi

  // Trimite temperatura și unitatea prin Bluetooth
  bt.print("Temp: ");
  bt.print(tempDisplay, 1);
  bt.print(" ");
  bt.print(unit);
  bt.print("\n");

  // Controlul LED-urilor și buzzer-ului în funcție de temperatura măsurată
  if (tempC < 30.0) {
    digitalWrite(ledBluePin, HIGH);    // Temperatura scăzută - LED albastru ON
    digitalWrite(ledYellowPin, LOW);
    digitalWrite(ledRedPin, LOW);
    digitalWrite(buzzerPin, LOW);
  } 
  else if (tempC >= 30.0 && tempC <= 32.5) {
    digitalWrite(ledBluePin, LOW);
    digitalWrite(ledYellowPin, HIGH);  // Temperatura medie - LED galben ON
    digitalWrite(ledRedPin, LOW);
    digitalWrite(buzzerPin, LOW);
  } 
  else if (tempC > 32.5) {
    digitalWrite(ledBluePin, LOW);
    digitalWrite(ledYellowPin, LOW);
    digitalWrite(ledRedPin, HIGH);     // Temperatura mare - LED roșu și buzzer ON
    digitalWrite(buzzerPin, HIGH);
  }

  delay(1000);                         // Așteaptă 1 secundă înainte de următoarea citire
}

// Funcție care procesează comenzile primite prin Bluetooth
void processCommand(String cmd) {
  cmd.trim();                          // Elimină spațiile de la început/sfârșit
  cmd.toUpperCase();                   // Converteste comanda la majuscule

  if (cmd == "SET_C") {
    useFahrenheit = false;            // Afișează în grade Celsius
  } 
  else if (cmd == "SET_F") {
    useFahrenheit = true;             // Afișează în Fahrenheit
  } 
}

// Funcție care afișează alerta când temperatura este prea mare
void showAlert(float tempC) {
  bt.print("ALERT! Temperature too high: ");
  bt.print(tempC);
  bt.print(" C\n");
     // Mesaj de alertă
}

// Funcție care elimină mesajul de alertă când temperatura revine la normal
void removeAlert() {
    lcd.setCursor(0, 1);
    lcd.print("                  ");   // Curăță linia 2 de pe LCD
}


