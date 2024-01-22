#include <QTRSensors.h>
#include <EEPROM.h>

// pins
const int m11Pin = 7;
const int m12Pin = 6;
const int m21Pin = 4;                                                               
const int m22Pin = 5;
const int m1Enable = 11;
const int m2Enable = 10;

// speed variables
int m1Speed = 0;
int m2Speed = 0;
int motorSpeed;


// motorSpeed = kp * p + ki * i + kd * d;
float kp = 7.7;
float ki = 0.0001;
// float ki = 0;
float kd = 14;

// values passed to the PID controller.
int p = 1;
int i = 0;
int d = 0;

// values used for computing the derivative. 
int error = 0;
int lastError = 0;


//base speed multiplier
const float multiplier = 0.85;

const int maxSpeed = 255;
const int minSpeed = -255;
const int baseSpeed = 255* multiplier;


// constants used for calibration.
const int calibrationCycles = 6;
const int sensorCalibrationBound = 700;

QTRSensors qtr;
const int sensorCount = 6;
int sensorValues[sensorCount];



bool isCalib = 1;


void calibrate() {

  int calibrationSpeed = 100, state = 0, i = 0;
  while (isCalib) {
    qtr.calibrate();
    qtr.read(sensorValues);

    if (state == 0) {
      if (sensorValues[0] < sensorCalibrationBound) {
        setMotorSpeed(calibrationSpeed, -calibrationSpeed);
      } else {
        state = (state + 1) % 2;
        i++
      }
    }
    else{
      if (sensorValues[5] < sensorCalibrationBound) {
        setMotorSpeed(-calibrationSpeed, calibrationSpeed);
      } else {
        state = (state + 1) % 2;
        i++
      }
    }

    if(i == calibrationCycles * 2){
      isCalib = 0;
    }
  }


}

void setup() {
  pinMode(m11Pin, OUTPUT);
  pinMode(m12Pin, OUTPUT);
  pinMode(m21Pin, OUTPUT);
  pinMode(m22Pin, OUTPUT);
  pinMode(m1Enable, OUTPUT);
  pinMode(m2Enable, OUTPUT);


  qtr.setTypeAnalog();
  qtr.setSensorPins((const uint8_t[]){ A0, A1, A2, A3, A4, A5 }, sensorCount);

  Serial.begin(9600);
  calibrate();

}

void loop() {

  pid(kp, ki, kd);

  computeMotorsSpeed();

  setMotorSpeed(m1Speed, m2Speed);

  // setMotorSpeed(200,200); -> test for motor orientation
}



void pid(float kp, float ki, float kd) {
  error = map(qtr.readLineBlack(sensorValues), 0, 5000, -50, 50);

  p = error;
  i = i + error;
  d = error - lastError;
  lastError = error;

  motorSpeed = kp * p + ki * i + kd * d;
}


void computeMotorsSpeed() {
  m1Speed = baseSpeed;
  m2Speed = baseSpeed;

  if (error < 0) {
    m1Speed += motorSpeed;
  } else if (error > 0) {
    m2Speed -= motorSpeed;
  }

  m1Speed = constrain(m1Speed, minSpeed, maxSpeed);
  m2Speed = constrain(m2Speed, minSpeed, maxSpeed);
}


void setMotorSpeed(int motor1Speed, int motor2Speed) {

  motor1Speed = -motor1Speed;
  if (motor1Speed == 0) {
    digitalWrite(m11Pin, LOW);
    digitalWrite(m12Pin, LOW);
    analogWrite(m1Enable, motor1Speed);
  } else {
    if (motor1Speed > 0) {
      digitalWrite(m11Pin, HIGH);
      digitalWrite(m12Pin, LOW);
      analogWrite(m1Enable, motor1Speed);
    }
    else{
      digitalWrite(m11Pin, LOW);
      digitalWrite(m12Pin, HIGH);
      analogWrite(m1Enable, -motor1Speed);
    }
  }
  if (motor2Speed == 0) {
    digitalWrite(m21Pin, LOW);
    digitalWrite(m22Pin, LOW);
    analogWrite(m2Enable, motor2Speed);
  } else {
    if (motor2Speed > 0) {
      digitalWrite(m21Pin, HIGH);
      digitalWrite(m22Pin, LOW);
      analogWrite(m2Enable, motor2Speed);
    }
    else{
      digitalWrite(m21Pin, LOW);
      digitalWrite(m22Pin, HIGH);
      analogWrite(m2Enable, -motor2Speed);
    }
  }
}
