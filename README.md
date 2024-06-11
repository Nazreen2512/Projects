
#include <Wire.h>           
#include <LiquidCrystal_I2C.h>    
LiquidCrystal_I2C lcd(0x27,16,2);   
#include <Adafruit_MPU6050.h>
#include <Adafruit_Sensor.h>
#include <Wire.h>
#include "ThingSpeak.h" 
#include <ESP8266WiFi.h>
char ssid[] = "vivo 1935";         
char pass[] = "25122003";    
int keyIndex = 0;            
WiFiClient  client;

unsigned long myChannelNumber =2521755;
const char * myWriteAPIKey = "M8PJNP889AUJF05P";

String myStatus = "";
Adafruit_MPU6050 mpu;

void setup(void) {
  lcd.init();
  lcd.init();
  lcd.clear();         
  lcd.backlight(); 
  Serial.begin(115200);
  LCD_WELCOME();
  ThingSpeak.begin(client);
  while (!Serial)
    delay(10); 

  Serial.println("Adafruit MPU6050 test!");

 
  if (!mpu.begin()) {
    
    Serial.println("Failed to find MPU6050 chip");
    while (1) {
      delay(10);
    }
  }
  Serial.println("MPU6050 Found!");

  mpu.setAccelerometerRange(MPU6050_RANGE_8_G);
  Serial.print("Accelerometer range set to: ");
  switch (mpu.getAccelerometerRange()) {
  case MPU6050_RANGE_2_G:
    Serial.println("+-2G");
    break;
  case MPU6050_RANGE_4_G:
    Serial.println("+-4G");
    break;
  case MPU6050_RANGE_8_G:
    Serial.println("+-8G");
    break;
  case MPU6050_RANGE_16_G:
    Serial.println("+-16G");
    break;
  }
  mpu.setGyroRange(MPU6050_RANGE_500_DEG);
  Serial.print("Gyro range set to: ");
  switch (mpu.getGyroRange()) {
  case MPU6050_RANGE_250_DEG:
    Serial.println("+- 250 deg/s");
    break;
  case MPU6050_RANGE_500_DEG:
    Serial.println("+- 500 deg/s");
    break;
  case MPU6050_RANGE_1000_DEG:
    Serial.println("+- 1000 deg/s");
    break;
  case MPU6050_RANGE_2000_DEG:
    Serial.println("+- 2000 deg/s");
    break;
  }

  mpu.setFilterBandwidth(MPU6050_BAND_21_HZ);
  Serial.print("Filter bandwidth set to: ");
  switch (mpu.getFilterBandwidth()) {
  case MPU6050_BAND_260_HZ:
    Serial.println("260 Hz");
    break;
  case MPU6050_BAND_184_HZ:
    Serial.println("184 Hz");
    break;
  case MPU6050_BAND_94_HZ:
    Serial.println("94 Hz");
    break;
  case MPU6050_BAND_44_HZ:
    Serial.println("44 Hz");
    break;
  case MPU6050_BAND_21_HZ:
    Serial.println("21 Hz");
    break;
  case MPU6050_BAND_10_HZ:
    Serial.println("10 Hz");
    break;
  case MPU6050_BAND_5_HZ:
    Serial.println("5 Hz");
    break;
  }

  Serial.println("");
  delay(100);
}

void loop() {
wifi_conn();
  /* Get new sensor events with the readings */
  sensors_event_t a, g, temp;
  mpu.getEvent(&a, &g, &temp);

  /* Print out the values */
  Serial.print("Acceleration X: ");
  Serial.print(a.acceleration.x);
  Serial.print(", Y: ");
  Serial.print(a.acceleration.y);
  Serial.print(", Z: ");
  Serial.print(a.acceleration.z);
  Serial.println(" m/s^2");
      lcd.clear();
     lcd.setCursor(0, 0);
     lcd.print("A_X:");
     lcd.setCursor(0, 1);
     lcd.print(a.acceleration.x);
     lcd.setCursor(5, 0);
     lcd.print("A_Y:");
     lcd.setCursor(6, 1);
     lcd.print(a.acceleration.y);
     lcd.setCursor(9, 0);
     lcd.print("A_Z:");
     lcd.setCursor(9, 1);
     lcd.print(a.acceleration.z);
     delay(2000);

  Serial.print("Rotation X: ");
  Serial.print(g.gyro.x);
  Serial.print(", Y: ");
  Serial.print(g.gyro.y);
  Serial.print(", Z: ");
  Serial.print(g.gyro.z);
  Serial.println(" rad/s");
  lcd.clear();
     lcd.setCursor(0, 0);
     lcd.print("R_X:");
     lcd.setCursor(0, 1);
     lcd.print(g.gyro.x);
     lcd.setCursor(5, 0);
     lcd.print("R_Y:");
     lcd.setCursor(6, 1);
     lcd.print(g.gyro.y);
     lcd.setCursor(9, 0);
     lcd.print("R_Z:");
     lcd.setCursor(9, 1);
     lcd.print(g.gyro.z);
    delay(2000);
  Serial.print("Temperature: ");
  Serial.print(temp.temperature);
  Serial.println(" degC");

  Serial.println("");
  lcd.clear();
     lcd.setCursor(2, 0);
     lcd.print("TEMPERATURE:");
     lcd.setCursor(3, 1);
     lcd.print(temp.temperature);
     lcd.setCursor(9, 1);
     lcd.print(" degC");
     ThingSpeak.setField(1, a.acceleration.x);
     ThingSpeak.setField(2, a.acceleration.y);
     ThingSpeak.setField(3, a.acceleration.z);
     ThingSpeak.setField(4, g.gyro.x);
     ThingSpeak.setField(5, g.gyro.y);
     ThingSpeak.setField(6, g.gyro.z);
     ThingSpeak.setField(7, temp.temperature);
     ThingSpeak.setStatus(myStatus);
  
 
  int x = ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);
  if(x == 200){
    Serial.println("Channel update successful.");
  }
  else{
    Serial.println("Problem updating channel. HTTP error code " + String(x));
  }
 
  delay(20000); 
  //delay(500);
}


void LCD_WELCOME()
{
  lcd.begin(16, 2);
  lcd.setCursor(6, 0);
  lcd.print("WELCOME");
   lcd.setCursor(2, 1);
  lcd.print("TO MY PROJECT");
  delay(500);
  
}
void wifi_conn()
{

if(WiFi.status() != WL_CONNECTED){
    Serial.print("Attempting to connect to SSID: ");
    Serial.println(ssid);
    while(WiFi.status() != WL_CONNECTED){
      WiFi.begin(ssid, pass);  // Connect to WPA/WPA2 network. Change this line if using open or WEP network
      Serial.print(".");
      delay(5000);     
    } 
    Serial.println("\nConnected.");
  }

}
