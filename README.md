Em um ambiente hospitalar, a gestão eficiente da temperatura desempenha um papel crucial na promoção do bem-estar dos pacientes e no funcionamento adequado de equipamentos médicos sensíveis. No entanto, os desafios relacionados à temperatura podem surgir, criando um ambiente propício a complicações.

Um dos principais problemas enfrentados pelos hospitais é a variação incontrolável de temperatura em diferentes áreas. As salas de cirurgia, por exemplo, demandam condições precisas para garantir a segurança dos procedimentos e o conforto dos profissionais de saúde. Enquanto isso, em alas de internação, manter um ambiente térmico adequado é vital para a recuperação eficaz dos pacientes.

A falta de controle térmico pode resultar em desconforto para os pacientes, aumentando o risco de complicações pós-operatórias e prolongando o tempo de recuperação. Além disso, equipamentos médicos sensíveis, como medicamentos e dispositivos eletrônicos, podem ser afetados negativamente por variações extremas de temperatura, comprometendo sua eficácia.

Solução

Criar um termômetro para saber a temperatura do ambiente e tentar controla-la usando outro dispositivos 

link para tinkercad: https://www.tinkercad.com/things/0heAjXqOErM-copy-of-termometro/editel?returnTo=%2Fdashboard&sharecode=cIkZ8-3S-Ih6C_QBNB9-yErSnrhZzypE-lKaEPXnZ64

código para o projeto funcionar:

// biblioteca lcd
#include <LiquidCrystal.h>

#define TO_CELSIUS(x) (x - 500)/10.0
#define TO_VOLT(x) x * (5000/1024.0)
#define TEMP_PIN A0
#define SERIAL 9600
#define BUTTON_PIN 8
#define LEDR_PIN 10
#define LEDG_PIN 9
#define LEDB_PIN 7
#define BUZZER_PIN 6

// inicialização do objeto com os pinos
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

// caracteres especiais p/ lcd
byte low[] = {
  B00000,
  B00100,
  B10101,
  B01110,
  B11011,
  B01110,
  B10101,
  B00100
};

byte ok[] = {
  B00000,
  B00001,
  B00011,
  B10110,
  B11100,
  B01000,
  B00000,
  B00000
};

byte warning[] = {
  B01110,
  B01110,
  B01110,
  B01110,
  B01110,
  B00000,
  B01110,
  B01110
};

byte critical[] = {
  B00000,
  B01110,
  B10101,
  B11011,
  B01110,
  B01110,
  B00000,
  B00000
};

byte degree[] = {
  B00111,
  B00101,
  B00111,
  B00000,
  B00000,
  B00000,
  B00000,
  B00000
};

// estado global de ON/OFF
int estado = HIGH;
int atual;
int anterior = LOW;
long time = 0;
long debounce = 100;

// cores do led
void led(int r, int g, int b) {
  digitalWrite(LEDR_PIN, r);
  digitalWrite(LEDG_PIN, g);
  digitalWrite(LEDB_PIN, b);
}

void setup() {
  Serial.begin(SERIAL);
  
  // setup do led
  pinMode(LEDR_PIN, OUTPUT);
  pinMode(LEDG_PIN, OUTPUT);
  pinMode(LEDB_PIN, OUTPUT); 
  
  // setup do botão
  pinMode(BUTTON_PIN, INPUT_PULLUP); 
  
  // setup do buzzer
  pinMode(BUZZER_PIN, OUTPUT);
   
  // setup do lcd
  lcd.begin(16, 2);
  lcd.setCursor(0, 0);
  lcd.print("Temp:");
  lcd.setCursor(0, 1);
  lcd.print("Status:");
  lcd.createChar(0, low);
  lcd.createChar(1, ok);
  lcd.createChar(2, warning);
  lcd.createChar(3, critical);
  lcd.createChar(4, degree);
}

void loop() {

  // seta o estado global de acordo com o botão
  atual = digitalRead(BUTTON_PIN);  
  if (atual == HIGH && anterior == LOW
      && millis() - time > debounce) {
    if (estado == HIGH)
      estado = LOW;
    else
      estado = HIGH;

    time = millis();    
  }
  
  if (estado){
    lcd.display();
    
    // dados de temperatura
    int leitura = analogRead(TEMP_PIN);
    float voltagem = TO_VOLT(leitura);
    float temperatura = TO_CELSIUS(voltagem);

    // printa o valor depois da string
    lcd.setCursor(5, 0);
    lcd.print(temperatura);
    lcd.write(byte(4));
    lcd.print("C        ");
    lcd.setCursor(7, 1);

    // printa diferentes alertas de acordo com a temperatura
    if(temperatura <= 20.0) {
      led(0,255,0);
      tone(BUZZER_PIN, 100);
      lcd.print("LOW");
      lcd.write(byte(0));
      lcd.print("            ");
    } else if (temperatura > 20.0 && temperatura <= 50.0) {
      led(0,0,255);
      noTone(BUZZER_PIN);
      lcd.print("OK");
      lcd.write(byte(1));
      lcd.print("             ");
    } else if (temperatura > 50.0 && temperatura <= 100.0) {
      led(255,0,255);
      tone(BUZZER_PIN, 1000);
      lcd.print("WARNING"); 
      lcd.write(byte(2));
      lcd.print("    ");
    } else {
      led(255,0,0);
      tone(BUZZER_PIN, 5000);
      lcd.print("CRITICAL");
      lcd.write(byte(3));
    } 
  } else {
  	lcd.noDisplay();
    led(0,0,0);
    noTone(BUZZER_PIN);
  } 
} 
