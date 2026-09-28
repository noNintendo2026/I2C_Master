## 📘 Teoría de Operación: Protocolo I2C y Memoria EEPROM

A continuación se detallan los fundamentos teóricos del bus I2C y las características de la memoria externa, información clave para el desarrollo de la máquina de estados de nuestro periférico Master.

### Características Generales del Bus I2C
*   **Tipo de Bus:** Bus de serie síncrono.
*   **Topología:** Multi-master / Multi-slave.
*   **Líneas físicas (2 hilos + Tierra):** 
    *   `SCL` (Serial Clock): Señal de reloj.
    *   `SDA` (Serial Data): Señal de datos bidireccional.
    *   `GND`: Tierra común.
*   **Capacidad:** Permite direccionar hasta 127 dispositivos esclavos en el mismo bus.

### Roles en la Comunicación
*   **Master (Maestro):** 
    *   Genera siempre la señal de reloj (`SCL`).
    *   Inicia y finaliza la comunicación.
    *   Envía la dirección del esclavo con el que desea hablar.
*   **Slave (Esclavo):**
    *   Solo responde si es llamado por su dirección.
    *   Nunca inicia la comunicación por sí mismo.

### Mecanismos de Control del Bus
*   **Arbitraje:** Si dos Masters intentan enviar datos a la vez, gana el que envía un `0` lógico. Mientras tanto, el otro Master detecta que el estado de la línea `SDA` no coincide con lo que está transmitiendo y se retira automáticamente.
*   **Clock Stretching (Estiramiento de reloj):** Un esclavo puede mantener la línea `SCL` en `0` (bajo) para pedirle más tiempo al Master antes de continuar.

---

### Estructura de la Trama de Escritura/Lectura
La transmisión de datos sigue una secuencia estricta basada en el estado de las líneas, las cuales se mantienen en estado alto (High) mediante resistencias *pull-up* cuando están en reposo.

1.  **Condición de START:**
    *   La línea `SDA` baja a `0` mientras `SCL` sigue en alto (`1`).
    *   Luego, el Master baja `SCL` para empezar a transmitir el primer byte.
2.  **Transmisión de Dirección y R/W:**
    *   El nivel de `SDA` solo puede cambiar cuando `SCL` está en bajo (`0`). 
    *   El receptor muestrea (lee) el bit cuando `SCL` está en alto (`1`). En este punto, `SDA` debe permanecer estable.
3.  **Reloj y Datos (Bits):**
    *   El reloj generado por el Master tiene **9 pulsos por byte**: 8 pulsos para los datos y 1 pulso extra para el bit de confirmación (ACK/NACK).
4.  **Confirmación (ACK / NACK):**
    *   Ocurre en el **9º pulso de SCL**.
    *   **ACK:** El Master suelta la línea `SDA` y el esclavo la lleva a `0` para confirmar la recepción.
    *   **NACK:** Si la línea `SDA` se queda en `1` (alto) durante el 9º pulso, significa que el esclavo no respondió o hubo un error.
5.  **Condición de STOP:**
    *   Es un flanco opuesto al START.
    *   `SCL` sube primero a `1`. Luego, mientras `SCL` está en alto, `SDA` sube a `1`.

---

### Interfaz con Memoria EEPROM
El dispositivo esclavo objetivo de este proyecto es una memoria EEPROM (Memoria no volátil basada en celdas de transistores de compuerta flotante que se escribe y borra eléctricamente; a diferencia de la Flash, que borra por bloques/sectores).

*   **Direccionamiento en el Bus I2C:**
    *   Formato del byte de control: `1010` (Fijo para EEPROM) + `A2 A1 A0` (Pines de hardware) + `R/W` (Bit de Lectura/Escritura).
    *   Los pines `A2-A0` permiten conectar hasta **8 memorias EEPROM** idénticas en el mismo bus.
*   **Direccionamiento Interno:**
    *   Se utiliza un *Word* para indicar la dirección interna de la celda donde se va a guardar/leer.
    *   La EEPROM cuenta con un puntero interno que se autoincrementa tras leer o escribir cada byte.
*   **Ciclo Interno de Escritura (Acknowledge Polling):**
    *   Al recibir una orden de escritura, la EEPROM inicia un ciclo interno de guardado (típicamente tarda ~5ms).
    *   Durante este ciclo, la EEPROM **no responde** al bus.
    *   El Master debe repetir constantemente las señales de `START` seguido de `Device Address + W` hasta que la EEPROM responda con un ACK, indicando que terminó de guardar.
