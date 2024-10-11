## Unidad 3
# Ejercicio 1
Un protocolo binario es un conjunto de reglas que definen cómo se debe estructurar y transmitir la información entre dispositivos utilizando datos binarios (0 y 1). se transmiten directamente secuencias binarias que representan comandos, respuestas y datos.

# 1. Cómo se ve un protocolo binario
Un protocolo binario define un conjunto de mensajes que contienen secuencias específicas de bits. Cada mensaje suele tener un formato estructurado que incluye partes que indican:

- Instrucciones que el sensor debe ejecutar.
- Datos que se desean leer o escribir.
- Respuestas del sensor.

# 2. Un mensaje en un protocolo binario típico puede tener las siguientes partes:

- Cabecera (Header): Generalmente, la primera parte del mensaje que contiene información sobre el tipo de mensaje o el inicio de la comunicación.

- Comando (Command Code): Indica la acción que se debe realizar. Por ejemplo, leer datos, escribir datos o cambiar configuraciones en el sensor.

- Datos (Data Fields): Estos son los valores binarios que representan la información que se transmite. Puede ser, por ejemplo, la lectura de una medición del sensor.

- Checksum o CRC (Cyclic Redundancy Check): Esta es una secuencia de bits que se utiliza para verificar la integridad de los datos durante la transmisión. Se usa para detectar errores.

- Terminator: A veces, se incluye un finalizador o delimitador que señala el final del mensaje.

# 3. Para qué sirve cada parte del mensaje

- Cabecera: Identifica el inicio del mensaje y ayuda a sincronizar la comunicación entre el emisor y el receptor. También puede especificar el tipo de mensaje o la versión del protocolo.

- Comando: Indica lo que el dispositivo debe hacer. Por ejemplo, en un sensor, un comando puede solicitar que envíe una medición o cambie a un modo de operación diferente.

- Datos: Contiene la información que se está transmitiendo, como una medida obtenida por el sensor o un valor que se quiere enviar al sensor para modificar su comportamiento.

- Checksum o CRC: Verifica que los datos recibidos no se hayan corrompido durante la transmisión. Es un mecanismo de control de errores.

- Terminator: Marca el final del mensaje para que el receptor sepa que ha recibido todo el mensaje completo.

## Ejercicio 2
La función `Serial.readBytesUntil()` se ha excluido porque, como mencionas, en un protocolo binario típicamente no existe un carácter delimitador explícito que indique el fin del mensaje como el `\n` en un protocolo basado en ASCII

## Ejercicio 3
El concepto de endian se refiere al orden en el que los bytes de una variable multibyte (por ejemplo, un entero o un número en punto flotante) se almacenan o se transmiten. Dependiendo del sistema, existen dos formas principales de organizar estos bytes: little endian y big endian.

# 1. Little Endian
En el formato little endian, el byte de menor peso, se almacena o se transmite primero. Es decir, los bytes se organizan en orden inverso, desde el más pequeño al más grande.

- Ej: Supongamos que tienes el número hexadecimal 0x12345678. En formato little endian, los bytes se almacenarían de la siguiente forma:

`kotlin`
- Byte 1: 0x78 (menor peso)
- Byte 2: 0x56
- Byte 3: 0x34
- Byte 4: 0x12 (mayor peso)
# 2. Big Endian
En el formato big endian, el byte de mayor peso se almacena o se transmite primero. Los bytes se organizan en el orden "natural" desde el más grande al más pequeño.

- Ej: Utilizando el mismo número 0x12345678, en formato big endian, los bytes se almacenarían de la siguiente forma:

`kotlin`
- Byte 1: 0x12 (mayor peso)
- Byte 2: 0x34
- Byte 3: 0x56
- Byte 4: 0x78 (menor peso)

## Ejercicio 4
¿En qué endian estamos transmitiendo el número?
En el código original, estás transmitiendo el número en little endian. Esto se debe a que la mayoría de los microcontroladores basados en arquitecturas como ARM (utilizados en placas Arduino) almacenan los datos en memoria en formato little endian de forma predeterminada.

- En little endian, el byte menos significativo (LSB) se transmite primero, y el byte más significativo (MSB) se transmite al final. Es decir, los bytes del número en punto flotante 3589.3645 (45 60 55 D5) se envían en el siguiente orden:

- D5 55 60 45
¿Cómo transmitir en el endian contrario (big endian)?
Para transmitir el número en big endian, debes enviar primero el byte más significativo (MSB) y luego el menos significativo (LSB). En este caso, simplemente inviertes el orden de los bytes antes de transmitirlos. Ya has dado una solución para esto con el siguiente código:
```
#include <Arduino.h>
void setup() {
    Serial.begin(115200);
}

void loop() {
    // 45 60 55 d5 //
    static float num = 3589.3645;
    static uint8_t arr[4] = {0};

    if (Serial.available()) {
        if (Serial.read() == 's') {
            memcpy(arr, (uint8_t *)&num, 4);

            // Transmitir en orden inverso (big endian)
            for (int8_t i = 3; i >= 0; i--) {
                Serial.write(arr[i]);
            }
        }
    }
}
```
- Diferencia en la transmisión:
Little Endian: Los bytes se envían en el orden D5 55 60 45.
Big Endian: Los bytes se envían en el orden 45 60 55 D5.
Al invertir el orden de los bytes, estás asegurando que el receptor pueda interpretar los datos en el formato adecuado, dependiendo de si espera little endian o big endian.

## Ejercicio 5
- Este código nos permite enviar dos números en ambos formatos endian y un tercer número adicional en little endian.
```
void setup() {
    Serial.begin(115200);
}

void loop() {
    // Definir tres números en punto flotante
    static float num1 = 3589.3645;
    static float num2 = 1234.5678;
    static float num3 = 8765.4321;

    // Crear buffers para almacenar los bytes de cada número
    static uint8_t arr1[4] = {0};
    static uint8_t arr2[4] = {0};
    static uint8_t arr3[4] = {0};

    if (Serial.available()) {
        // Leer un carácter del puerto serial
        if (Serial.read() == 's') {
            // Copiar los números a sus respectivos buffers en formato IEEE 754
            memcpy(arr1, (uint8_t *)&num1, 4);
            memcpy(arr2, (uint8_t *)&num2, 4);
            memcpy(arr3, (uint8_t *)&num3, 4);

            // Enviar num1 en formato little endian (por defecto)
            Serial.write(arr1, 4);
            
            // Enviar num2 en formato big endian
            for (int8_t i = 3; i >= 0; i--) {
                Serial.write(arr2[i]);
            }

            // Enviar num3 en formato little endian (por defecto)
            Serial.write(arr3, 4);
        }
    }
}
```