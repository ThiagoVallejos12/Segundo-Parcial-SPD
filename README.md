# _Segundo parcial SPD_

Nombre y apellido: Thiago Vallejos

Division: J

# Sistema antiincendio

![Captura de pantalla (107)](https://github.com/ThiagoVallejos12/Segundo-Parcial-SPD/assets/108820694/e0eeb6d6-f869-40d3-9421-4167d1c83bbe)

## Descripcion del proyecto

Este proyecto simula un sistema antiincendio.

En el proyecto se utiliza:

-1 Placa arduino.

-1 Breadboard.

-1 LCD 16 x 2

-1 IR Remote

-1 IR Sensor

-1 Micro servo

-1 TMP36

-2 Leds(Rojo y verde)

-3 Resistencias

## Como funciona

Este sistema va a estar constantemente pendiente de que el sensor IR reciba una señal del control remoto con el codigo del botonPower.

En caso de que se reciba esa señal, se empezara a calcular constantemente la temperatura, mediante el sensor de temperatura (TMP36) y se mostrara en el LCD la temperatura y la estacion del año.

Si la temperatura esta por debajo de los 40 grados, estara encendida permanentemenete la luz verde. En caso de superar los 40 grados, los leds parpadearan, dando a entender que hay una temperatura elevada. Cuanta mas temperatura haya, con mas frecuencia parpadearan.

En caso de que la temperatura supere los 60 grados, se habilitara el sistema antiincendios. El servo se girara en 90 grados y se avisara por el display que hay un incendio. 

Tambien, el led rojo parpadeara constantemente.

El sistema se puede encender y apagar siempre que quieras desde el control IR.

## Defines, declaración de funciones y declaración de varibables.
```c++
#include <Servo.h>
#include <LiquidCrystal.h>
#include <IRremote.h>

#define LEDROJO 12
#define LEDVERDE 13
#define RS 2
#define E 3
#define D4 4
#define D5 5 
#define D6 6
#define D7 7
#define TERMOMETRO A0
#define SERVO 9
#define CONTROL 11

#define BotonPower 0xFF00BF00
#define BotonOn 0xEF10BF00
#define BotonOff 0xEE11BF00

bool detectarBoton(IRrecv detector, uint32_t boton);
void sistemaIncendio(int estado, int led1, int led2, Servo servo, int temperatura);
void verificarEstacion(int temperatura, LiquidCrystal lcd);

int lecturaTemperatura = 0;
bool estadoSistema = false;

LiquidCrystal miLcd(RS, E, D4, D5, D6, D7);
IRrecv detectorIR(CONTROL);
Servo miServo;
```

## Setup
```c++
pinMode(LEDROJO, OUTPUT);
pinMode(LEDVERDE, OUTPUT);
Serial.begin(9600);
miLcd.begin(16,2);
detectorIR.begin(CONTROL, DISABLE_LED_FEEDBACK);
miServo.attach(SERVO);
```

## Código principal
Se limpia la pantalla del LCD dando a entender que esta apagado. Se mueve el servo a la posicion en la que esta cerrado el sistema antiincendio

Se verifica constantemente si se presiona el BotonPower del control. En caso de ser presionado se ejecuta el ciclo while.

El ciclo while va a verificar constantemente si la temperatura es mayor a 60. En caso de serlo, se llama a la funcion sistemaIncendio, la cual activara el sistema de incendio.

```c++
miLcd.clear();
estadoSistema = detectarBoton(detectorIR, BotonPower);
sistemaIncendio(2, LEDROJO, LEDVERDE, miServo, lecturaTemperatura);
while (estadoSistema)
{
  lecturaTemperatura = map(analogRead(A0),20,358,-40,125);
  miLcd.setCursor(0,0);
  miLcd.print("TEMPERATURA:");
  miLcd.print(lecturaTemperatura);
  miLcd.print("C ");
  verificarEstacion(lecturaTemperatura, miLcd);
  if (lecturaTemperatura >= 60)
  {
    sistemaIncendio(1, LEDROJO, LEDVERDE, miServo, lecturaTemperatura);
  }
  else
  {
    sistemaIncendio(0, LEDROJO, LEDVERDE, miServo, lecturaTemperatura);
  }
  
  estadoSistema = !detectarBoton(detectorIR, BotonPower);
}
```

## Codigo de funciones.
### *"verificarEstacion"*

Esta función verifica la estación del año según la temperatura actual. Recibe la temperatura y un objeto LiquidCrystal como argumentos. 

Imprime el nombre de la estación en la segunda línea de la pantalla LCD según la temperatura.

```c++
void verificarEstacion(int temperatura, LiquidCrystal lcd)
{
  if (temperatura < 60)
  {
     if (temperatura < 10)
    {
      lcd.setCursor(0,1);
      lcd.print("INVIERNO        ");
    }
    else if(temperatura < 15)
    {
      lcd.setCursor(0,1);
      lcd.print("OTONO           ");
    }
    else if(temperatura < 20)
    {
      lcd.setCursor(0,1);
      lcd.print("PRIMAVERA       ");
    }
    else
    {
      lcd.setCursor(0,1);
      lcd.print("VERANO          ");
    } 
  }
}
```

### *"detectarBoton"*
Esta función se utiliza para detectar si se ha presionado un botón en el control remoto infrarrojo. 

Recibe un objeto IRrecv y un código de botón como argumentos. Devuelve true si se detecta el botón especificado, de lo contrario, devuelve false.

```c++
bool detectarBoton(IRrecv detector, uint32_t boton)
{
  int retorno = false;
  if (detector.decode())
  {
    if (detector.decodedIRData.decodedRawData == boton)
    {
      retorno = true;
      delay(100);
    }
    detector.resume();
  }
  return retorno;
}
```

### *"sistemaIncendio"*

Esta función controla el sistema de incendio. Recibe el estado actual del sistema (0, 1 o 2), los pines de los LEDs (led1 y led2), un objeto Servo y la temperatura como argumentos. 

Dependiendo del estado, se realizan diferentes acciones, como encender o apagar los LEDs y mover el servo.

```c++
void sistemaIncendio(int estado, int led1, int led2, Servo servo, int temperatura)
{
  digitalWrite(led2, LOW);
  digitalWrite(led1, LOW);
  switch (estado)
  {
    case 0:
    	servo.write(0);
    	if (temperatura < 40)
        {
          digitalWrite(led2, HIGH);
        }
    	else if (temperatura < 50)
        {
          digitalWrite(led2, HIGH);
    	  digitalWrite(led1, LOW);
          delay(250);
          digitalWrite(led2, LOW);
    	  digitalWrite(led1, HIGH);
          delay(250);
        }
    	else if (temperatura < 55)
        {
          digitalWrite(led2, HIGH);
    	  digitalWrite(led1, LOW);
          delay(175);
          digitalWrite(led2, LOW);
    	  digitalWrite(led1, HIGH);
          delay(175);
        }
    	else if (temperatura < 60)
        {
          digitalWrite(led2, HIGH);
    	  digitalWrite(led1, LOW);
          delay(100);
          digitalWrite(led2, LOW);
    	  digitalWrite(led1, HIGH);
          delay(100);
        }
  	break;
    case 1:
    	servo.write(90);
        miLcd.setCursor(0,1);
        miLcd.print("ALERTA: INCENDIO!");
    	digitalWrite(led1, HIGH);
    	delay(100);
    	digitalWrite(led1, LOW);
    	delay(100);
    break;
    case 2:
    	servo.write(0);
    break;
  }
}
```

## Proyecto en tinkercad
[Segundo parcial](https://www.tinkercad.com/things/k8Xg7A7mKyn)
