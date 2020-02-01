https://tutorials-raspberrypi.com/expand-raspberry-pi-gpios-with-i2c-port-expander/

Ampliar los GPIO de Raspberry Pi con I2C Port Expander.

<img src="https://tutorials-raspberrypi.de/wp-content/uploads/2014/03/mcp23017_pinout-300x243.png" 
alt ="mcp23017_pinout" witdh="300">

Una pequeña explicación de los pines principales:

    GPA0-7 y GPB0-7 son los pines GPIO
    
    A0, A1, A2 están conectados a + (3.3V) o - (GND) y definen el nombre internamente. 
    
    Si se conectan varios expansores de puertos, cada uno debe ser claramente identificable. 
    
    Con el primer expansor I²C conectarías todo a GND
    
    el siguiente expansor A0 a 3.3V y los otros dos a GND. 
    
    En el tercer expansor A1 a 3.3V y los otros dos a GND, etc. 
    
    Por lo tanto, es posible hasta 2³ y conectar un expansor de 8 puertos.
    
    VDD (Pin 9) obtiene el voltaje de entrada (3.3V)
    
    VSS (pin 10) está conectado a GND
    
    SCL (pin 12) está conectado al pin GPIO 5 del Pi
    
    SDA (pin 13) está conectado al GPIO pin 3 del Pi
    
Se construirá un circuito con 3 LED (como resistencias en serie de 330Ω).

<img src="https://tutorials-raspberrypi.com/wp-content/uploads/2014/03/i2c_Steckplatine1-768x923.png" 
alt ="conexiones" witdh="768">


Lo primero que debe hacer es desbloquear el I2C en el Pi. La forma más fácil de hacerlo es mediante

  sudo raspi-config 

Se activa en "Opciones avanzadas"> "I2C".
Para versiones anteriores de Raspbian, también debe editar un archivo

  sudo nano / etc / modules 

y agrega estas dos líneas al final:

  i2c-bcm2708
 i2c-dev 

Guarde y salga con CTRL + O y CTRL + X.

Ahora los módulos deben eliminarse del archivo de la lista negra, de lo contrario, no funcionarán.

  sudo nano /etc/modprobe.d/raspi-blacklist.conf 

y poner un # delante de las dos entradas.

  #blacklist spi-bcm2708
 #blacklist i2c-bcm2708 

Guarde nuevamente con CTRL + O y CTRL + X y salga.

Para que podamos abordar el I2C ahora, tenemos que instalar algunos paquetes más.

  sudo apt-get install python-smbus i2c-tools 

Luego apague el Pi, espere unos segundos y desconéctelo de la corriente. 

Después de que todo esté conectado y todas las conexiones hayan sido revisadas nuevamente, 
inicie el Pi y espere hasta que se haya iniciado.

Utilizo una Raspberry Pi Rev.2, así que la pruebo con:

  sudo i2cdetect -y 1 

Si tiene un Pi Rev.1, debe ingresar 0 en lugar de 1. La salida se ve así: 

pi@raspberrypi ~ $ sudo i2cdetect -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: 20 -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --

--------


La dirección 0x20 (hexadecimal) contiene el I2C. Si A2 estuviera conectado, por ejemplo, a 3.3V (A1 y A0 a GND), sería direccionable en la dirección 0x24. Esto, como se mencionó anteriormente, es importante si ha conectado varios expansores de puertos para poder abordarlos claramente.

Para direccionar los LED, los puertos deben declararse como entrada o salida (Rev1, vuelva a ajustar el usuario).

  sudo i2cset -y 1 0x20 0x01 0x00 

Aquí hay algunos ejemplos que explican cómo funciona el comando:

  i2cset -y 1 0x20 0x01 0x00 #todos los pines de GPB salen
 i2cset -y 1 0x20 0x01 0x04 # GPB2 es entrada, el resto de la salida GPB (desde 0x04 en binario 00000100)
 i2cset -y 1 0x20 0x00 0x80 # GPA7 es entrada, el resto de salida GPA 

En primer lugar, se trata la dirección dirigida por i2cdetect . El segundo valor está en esta tabla (de la hoja de datos ):

<img src="https://tutorials-raspberrypi.com/wp-content/uploads/2014/03/register-768x524.jpg" alt ="tabla" witdh="768">

----

Entonces, después de haber especificado la dirección (IODIRB) (0 = Salida, 1 = Entrada), queremos dejar que los tres LED se iluminen (Binario 00000111 = 0x07):

  sudo i2cset -y 1 0x20 0x15 0x07 

Si tuviéramos que usar los pines GPA, en lugar de 0x15, sería 0x14.
Para que los LED dejen de encenderse, debemos restablecer el nivel de los pines a 0:

  sudo i2cset -y 1 0x20 0x15 0x00 
  
  Raspberry Pi MCP23017 Python Script para entrada y salida

Entonces creamos un script

  sudo nano i2c_input_output.py 

con el siguiente contenido: 

<pre><code>
import smbus
import time
 
#bus = smbus.SMBus(0) # Rev 1 Pi
bus = smbus.SMBus(1) # Rev 2 Pi
 
DEVICE = 0x20 # Device Adresse (A0-A2)
IODIRA = 0x00 # Pin Register fuer die Richtung
IODIRB = 0x01 # Pin Register fuer die Richtung
OLATB = 0x15 # Register fuer Ausgabe (GPB)
GPIOA = 0x12 # Register fuer Eingabe (GPA)
 
# Define GPA pin 7 as input (10000000 = 0x80)
# Binary: 0 means output, 1 means input
bus.write_byte_data(DEVICE,IODIRA,0x80)
 
# Define all GPB pins as output (00000000 = 0x00)
bus.write_byte_data(DEVICE,IODIRB,0x00)
 
# Set all 7 output bits to 0
bus.write_byte_data(DEVICE,OLATB,0)
 
# Function that makes all LEDs light up.
def aufleuchten():
    for MyData in range(1,8):
        # Count from 1 to 8, which is binary
        # from 001 to 111.
        bus.write_byte_data(DEVICE,OLATB,MyData)
        print "Zahl:", MyData, "Binaer:", '{0:08b}'.format(MyData)
        time.sleep(1)
        # Reset all pins to 0 again
        bus.write_byte_data(DEVICE,OLATB,0)
 
# Endless loop waiting at the push of a button
while True:
    # Read status of GPIOA register
    Taster = bus.read_byte_data(DEVICE,GPIOA)
 
    if Taster & 0b10000000 == 0b10000000:
        print "Taster gedrueckt"
        aufleuchten()
       
</code></pre>

Guarde y salga con CTRL + O y CTRL + X.

Para comenzar el script ahora, ingresamos

  sudo python i2c_input_output.py 

Tan pronto como presione el botón, los LED se encenderán. Al presionar CTRL + C, puede cancelar la secuencia de comandos y volver a la consola.

Como puede ver, usarlo es bastante fácil y ha creado otros 16 pines GPIO. 
