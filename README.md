#include <AFMotor.h> // motor shield library
#include <Servo.h>
char bt;   //value sent over via bluetooth
int vel = 150;
long duration,cm;
byte safeDistance=40;
Servo myservo;
int pos = 150;
#define trigPin 32 // define the pins of your sensor
#define echoPin 33
#define light_FR  A0    //LED front    pin A0 for Arduino Uno
#define light_BK   A1    //LED back     pin A1 for Arduino Uno
#define RIGHT A3  // Right IR sensor connected to analog pin A2 of Arduino Uno:
#define LEFT A4
#define warning A15
bool warn = false;
boolean lightFront = false;
boolean lightBack = false; 
bool Right_Value = 0; //Variable to store Right IR sensor value:
bool Left_Value = 0;  //Variable to store Left IR sensor value:
bool mode = false;
bool mode1 = false;
AF_DCMotor motor1(1, MOTOR12_1KHZ); // set up motors.
AF_DCMotor motor2(2, MOTOR12_1KHZ);
AF_DCMotor motor3(3, MOTOR34_1KHZ);
AF_DCMotor motor4(4, MOTOR34_1KHZ);
int Left,Right,L,R;


void setup()
        { 
            Serial.begin(9600); 
            Serial2.begin(9600);
            myservo.attach(10);
            pinMode(light_FR, OUTPUT);
            pinMode(light_BK, OUTPUT);
            pinMode(warning, OUTPUT);
            pinMode(trigPin, OUTPUT); 
            pinMode(echoPin, INPUT);
            pinMode(RIGHT, INPUT); //set analog pin RIGHT as an input:
            pinMode(LEFT, INPUT);  //set analog pin RIGHT as an input:
            myservo.write(150); 
        }
 
 
void loop() {
  if (Serial2.available())  //if there is data being recieved
  {
    bt = Serial2.read();  //read it to select mode
  }


  if (mode == true && mode1 == false)
 {
    followHuman();
  }
   else if (mode1 == true && mode == false)
  {
    HandleUltrasonicAvoidance();
  }
  else
  {
    control();
  }

  switch (bt) {
    case 'X': mode = true; break;
    case 'V': mode1 = true; break;
    case 'x': mode = false; break;
    case 'v': mode1 = false; break;
     
      case 'W':lightFront = true;break;
      case 'w':lightFront = false;break;
      case 'U':lightBack = true;break;
      case 'u':lightBack = false;break;
  }
}
//mode1
void control()
          {
             servomotor();              //to rotate servoMotor
             blutooth();
             if (warn) {digitalWrite(warning, HIGH);}
             if (!warn) {digitalWrite(warning, LOW);}
             if (lightFront) {digitalWrite(light_FR, HIGH);}
             if (!lightFront) {digitalWrite(light_FR, LOW);}
             if (lightBack) {digitalWrite(light_BK, HIGH);}
             if (!lightBack) {digitalWrite(light_BK, LOW);}
             if (cm<=safeDistance)
            {
             warn=true;
            }
            else
            {
            warn=false;
            }

            
             
          }

 void servomotor()  //son sensor 
    {
           myservo.write(pos);
           cm = getDistance();     //get distance from the measurement functions 
	  
        if (cm<=safeDistance)
        {
          if (bt=='B'  && pos==150)  //if robot is stop and received B do rotate servo to 180 
            { 
              pos=0;
              myservo.write(pos);
			        cm = getDistance();
            }
            else if (bt=='F'  && pos==0)  //if robot is stop and received F do rotate servo to 0 
            { 
                pos=150;
                myservo.write(pos);
			      	  cm = getDistance();
                 
            }

        }
    }
  
void blutooth()
  {
    cm = getDistance();
    if (bt=='F' && cm>safeDistance)
    {
      myservo.write(150);
      pos=150;
      forward();
      cm = getDistance();
    }
        
    
    else if (bt=='B' && cm>safeDistance)
    {     
    myservo.write(0);
    pos=0;
    back();
    cm = getDistance();
    } 
    
   else if (bt=='L')  //left
    {
      left();
    }
  
   else if (bt=='R')  //right
    {
     right();
    }

   else if (bt=='I')  //  turn forwardRight
    {
      forwardright();
    }
  
    else if (bt=='J')  //  turn backright
    {
     backright();   
    }
  
    else if (bt=='G')  //  turn forwardleft
    {
      forwardleft();   
    }

    else if (bt=='H')  //  turn backleft
    {
      backleft();    
    }
  
    else if (bt=='S')  // Stop
    {
     Stop();
    }
    //to set speed

    else if (bt == '0'){vel = 0;}
    else if (bt == '1'){vel = 25;}
    else if (bt == '2'){vel = 50;}
    else if (bt == '3'){ vel = 75;}
    else if (bt == '4'){vel = 100;}
    else if (bt == '5'){vel = 125;}
    else if (bt == '6'){vel = 150;}
    else if (bt == '7'){vel = 175;}
    else if (bt == '8'){vel = 200;}
    else if (bt == '9'){vel = 225;}
    else if (bt == 'q'){vel = 255;}



    else{
       Stop(); 
       }
  }
	
    

//mode2
void followHuman()
{
   vel=130;
   Right_Value = digitalRead(RIGHT);             // read the value from Right IR sensor:
   Left_Value = digitalRead(LEFT);               // read the value from Left IR sensor:
   cm = getDistance(); 
   if(cm > 21 && cm < 60)
   {            
     forward();
   }
   else  if(cm > 10 && cm < 20)
   {            
     back();
   }

   else if(Right_Value==0 && Left_Value==1) 
   {       
     left();                                               
   }

   else if(Right_Value==1  &&  Left_Value==0)
   { 
     right();
   }

   else
   {                       
     Stop();
   }
  } 

void HandleUltrasonicAvoidance()
{

  cm = getDistance();
  if (cm <= 30)
  {
    Stop();
    back();
    delay(200);
    Stop();
    L = lookLeft();
    delay(500);
    myservo.write(pos);
    R = lookRight();
    delay(500);
    myservo.write(pos);
    if (L < R) {
	  vel=130;
      right();
      delay(800);
      Stop();
      delay(200);
    
     } 
	 else if (L > R)
	 {
	  vel=130;
      left();
      delay(800);
      Stop();
      delay(200);
    }
   }
   else
   {
     vel=150; 
		 forward();
         
  }




}


int lookLeft()
{
    myservo.write(180); 
    delay(500);
    Left = getDistance();
    return Left;


}


int lookRight()
{
    myservo.write(75); 
    delay(500);
    Right = getDistance();
    return Right;

}






     int getDistance()                                   //Measure the distance to an object
        {
              
              digitalWrite(trigPin, LOW);
              delayMicroseconds(2);
              digitalWrite(trigPin, HIGH);
              delayMicroseconds(10);
              digitalWrite(trigPin, LOW);
              duration = pulseIn(echoPin, HIGH);   // get a round trip time (microsecond)
              cm = duration/29/2;                  //git distanse by multi duration with speed of sound after convert it to (cm/microsecond) 
              return cm;                           //(1/29 = 0.0344 = 340*100/10^6)
        }





void forward()
{
  motor1.setSpeed(vel); //Define maximum velocity
  motor1.run(FORWARD); //rotate the motor clockwise
  motor2.setSpeed(vel); //Define maximum velocity
  motor2.run(FORWARD); //rotate the motor clockwise
  motor3.setSpeed(vel);//Define maximum velocity
  motor3.run(FORWARD); //rotate the motor clockwise
  motor4.setSpeed(vel);//Define maximum velocity
  motor4.run(FORWARD); //rotate the motor clockwise
}

void back()
{
  motor1.setSpeed(vel); //Define maximum velocity
  motor1.run(BACKWARD); //rotate the motor anti-clockwise
  motor2.setSpeed(vel); //Define maximum velocity
  motor2.run(BACKWARD); //rotate the motor anti-clockwise
  motor3.setSpeed(vel); //Define maximum velocity
  motor3.run(BACKWARD); //rotate the motor anti-clockwise
  motor4.setSpeed(vel); //Define maximum velocity
  motor4.run(BACKWARD); //rotate the motor anti-clockwise

}
void left()
{
  motor1.setSpeed(vel); //Define maximum velocity
  motor1.run(BACKWARD); //rotate the motor anti-clockwise
  motor2.setSpeed(vel); //Define maximum velocity
  motor2.run(BACKWARD); //rotate the motor anti-clockwise
  motor3.setSpeed(vel); //Define maximum velocity
  motor3.run(FORWARD);  //rotate the motor clockwise
  motor4.setSpeed(vel); //Define maximum velocity
  motor4.run(FORWARD);  //rotate the motor clockwise
}

void right()
{
  motor1.setSpeed(vel); //Define maximum velocity
  motor1.run(FORWARD); //rotate the motor clockwise
  motor2.setSpeed(vel); //Define maximum velocity
  motor2.run(FORWARD); //rotate the motor clockwise
  motor3.setSpeed(vel); //Define maximum velocity
  motor3.run(BACKWARD); //rotate the motor anti-clockwise
  motor4.setSpeed(vel); //Define maximum velocity
  motor4.run(BACKWARD); //rotate the motor anti-clockwise
}

void forwardright()
{
  motor1.setSpeed(vel); //Define maximum velocity
  motor1.run(FORWARD); //rotate the motor clockwise
  motor2.setSpeed(vel); //Define maximum velocity
  motor2.run(FORWARD); //rotate the motor clockwise
  motor3.setSpeed(vel/4); //Define 0.75% of maximum velocity
  motor3.run(FORWARD); //rotate the motor anti-clockwise
  motor4.setSpeed(vel); //Define maximum velocity
  motor4.run(FORWARD); //rotate the motor anti-clockwise
}

void forwardleft()
{
  motor1.setSpeed(vel); //rotate the motor clockwise
  motor2.setSpeed(vel/4); //Define 0.75% of maximum velocity
  motor1.run(FORWARD); //Define maximum velocity
  motor2.run(FORWARD); //rotate the motor clockwise
  motor3.setSpeed(vel); //Define maximum velocity
  motor3.run(FORWARD); //rotate the motor anti-clockwise
  motor4.setSpeed(vel); //Define maximum velocity
  motor4.run(FORWARD); //rotate the motor anti-clockwise
}

void backright()
{
  motor1.setSpeed(vel); //Define maximum velocity
  motor1.run(BACKWARD); //rotate the motor clockwise
  motor2.setSpeed(vel); //Define maximum velocity
  motor2.run(BACKWARD); //rotate the motor clockwise
  motor3.setSpeed(vel); //Define maximum velocity
  motor3.run(BACKWARD); //rotate the motor anti-clockwise
  motor4.setSpeed(vel/4); //Define 0.75% of maximum velocity
  motor4.run(BACKWARD); //rotate the motor anti-clockwise
}

void backleft()
{
  motor1.setSpeed(vel); //Define maximum velocity
  motor1.run(BACKWARD); //rotate the motor clockwise
  motor2.setSpeed(vel/4); //Define 0.75% of maximum velocity
  motor2.run(BACKWARD); //rotate the motor clockwise
  motor3.setSpeed(vel); //Define maximum velocity
  motor3.run(BACKWARD); //rotate the motor anti-clockwise
  motor4.setSpeed(vel); //Define maximum velocity
  motor4.run(BACKWARD); //rotate the motor anti-clockwise
}

void Stop()
{
motor1.setSpeed(vel); //Define minimum velocity
motor1.run(RELEASE); //stop the motor when release the button
motor2.setSpeed(vel); //Define minimum velocity
motor2.run(RELEASE); //rotate the motor clockwise
motor3.setSpeed(vel); //Define minimum velocity
motor3.run(RELEASE); //stop the motor when release the button
motor4.setSpeed(vel); //Define minimum velocity
motor4.run(RELEASE); //stop the motor when release the button
}
