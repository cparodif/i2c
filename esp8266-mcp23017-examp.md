# i2c

rpi arduino esp8266 i2c

http://www.esp8266learning.com/esp8266-mcp23017-example.php 
El mcp23017 es un expansor de puerto de E / S paralelo de propósito general de 16 bits para aplicaciones de bus I2C.

El puerto de E/S de 16 bits consta funcionalmente de dos puertos de 8 bits (PORTA y PORTB). El MCP23017 se puede configurar para funcionar en modos de 8 bits o 16 bits. Veamos el pinout en la imagen 
 <img src="https://i1.wp.com/www.esp8266learning.com/wp-content/uploads/2017/12/mcp23017-pinout-500x500.jpg" alt="pinout" height="400" width="400"> 

El MCP23017 funciona bien con 3.3v. Entonces conectamos Vdd al terminal 3v3 del módulo ESP8266 y, por supuesto, conectamos Vss a tierra.
Los pines GPB0-GPB7 y GPA0-GPA7 son los 16 puertos de E/S.
NC no está conectado.
SCL es la línea del reloj serial. Esto se conecta al pin analógico 5 en el arduino.
SDA es la línea de datos en serie. Se conecta al pin analógico 4 en el arduino.
INTA e INTB son pines de interrupción para las salidas. No estamos usando estos aquí.
El pin RESET es si desea que todas las salidas se restablezcan a 0. Conectaremos esto a + 5V.
A0, A1 y A2 son los pines de dirección. Esto es algo clave con este dispositivo, de hecho, puede tener 8 de estos conectados si usa una dirección diferente cada vez, esto, por supuesto, daría una gran cantidad de salidas potencialmente.

Aquí hay una tabla de las combinaciones de direcciones, tendremos A0, A1 y A2 vinculados a 0v, por lo que significa su dirección 0x20 

<img src="https://i0.wp.com/www.esp8266learning.com/wp-content/uploads/2017/12/MCP23017-addresspins1.jpg?w=537" alt="direcciones" height="328" width="484"> 

Esquema

No hemos mostrado el microcontrolador aquí, esto muestra solo las salidas LED, simplemente conecte sus pines ESP8266 I2C a los pines mcp23017 correspondientes

Obviamente, también puede usar portb, pero necesitaría cambiar el ejemplo de código 

<img src="https://i1.wp.com/www.esp8266learning.com/wp-content/uploads/2017/12/mcp23017-and-8-leds_schem.png" alt="conexiones"  width="800"> 

Código

No estamos utilizando ninguna biblioteca aquí, un ejemplo simple en el que básicamente envía un byte de datos: estamos enviando AA y luego 55 

<pre><code>
#include "Wire.h"
 
void setup()
{
Wire.begin(); // wake up I2C bus
// set I/O pins to outputs
Wire.beginTransmission(0x20);
Wire.write(0x00); // IODIRA register
Wire.write(0x00); // set all of port A to outputs
Wire.endTransmission();
}
 
void loop()
{
Wire.beginTransmission(0x20);
Wire.write(0x12); // address bank A
Wire.write((byte)0xAA); // value to send
Wire.endTransmission();
delay(500);
Wire.beginTransmission(0x20);
Wire.write(0x12); // address bank A
Wire.write((byte)0x55); // value to send
Wire.endTransmission();
delay(500);
}
</code></pre>
