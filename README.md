# AIML-Enabled-Battery-Management-System-for-Electric-vehicles
#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x27,16,2);

#include <DHT.h>  
#define DHTPIN 12
#define DHTTYPE DHT11 
DHT dht(DHTPIN, DHTTYPE);

float h;
float f;

String power;

int sensitivity = 66;
int adcValue= 0;
int offsetVoltage = 2500;
double adcVoltage = 0;
double currentValue = 0;
int relay = 9;
int limit = 6;
int buz = 7; 
float vOUT = 0.0;
float vIN = 0.0;
float R1 = 30000.0;
float R2 = 7500.0; 
float value;
 

#define voltageSensor A0
#define currentSensor A1

void setup() 
{
 Serial.begin(9600);
 dht.begin();      
 lcd.init();
 lcd.backlight();
 
 pinMode(voltageSensor,INPUT);
 pinMode(currentSensor,INPUT);
  pinMode(limit,INPUT_PULLUP);
   pinMode(buz,OUTPUT);
     pinMode(relay,OUTPUT);
}
 
void temp() 
{
    h = dht.readHumidity();
    f = dht.readTemperature(); 
   // String state = "T:"+String((float)f)+"c   ";
    String state = "T:"+String((float)f)+" H:"+String((float)h)+"C     ";
 
    lcd.setCursor(0,1);
    lcd.print(state);
}
void read_vol()
{
  adcValue = analogRead(currentSensor);
  adcVoltage = (adcValue / 1024.0) * 5200;
  currentValue = ((adcVoltage - offsetVoltage) / sensitivity); 

  value = analogRead(voltageSensor);
  vOUT = ((value * 5.0) / 1024.0);
  vIN = vOUT / (R2/(R1+R2));
 
  String power = "V:" +String(float(vIN))+ "V C:" + String(double(currentValue))+ "A";
  lcd.setCursor(0,0);
  lcd.print(power);

  if(vIN<4)
  {
   digitalWrite(relay,LOW);
   digitalWrite(buz,HIGH); 
   delay(500);
   digitalWrite(buz,LOW);
      delay(500);
         digitalWrite(buz,HIGH); 
   delay(500);
   digitalWrite(buz,LOW);
      delay(500);
         digitalWrite(buz,HIGH); 
   delay(500);
   digitalWrite(buz,LOW);
      delay(500);
    }
  else
  {
     digitalWrite(relay,HIGH);
   
    }
}

void ACC()
{
 int val = digitalRead(limit);
 if(val == 0)
 {
  digitalWrite(buz,HIGH);
  }
 else
 {
    digitalWrite(buz,LOW);
  }
} 
void loop()
{
 temp(); 
 read_vol();
 ACC();
}
