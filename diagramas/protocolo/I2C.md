### Descripción de la Figura 24-2 – Interconexión típica del bus I2C

La Figura 24-2 muestra la conexión básica entre un dispositivo **Master**, en este caso el controlador, y una memoria **EEPROM**, que funciona como dispositivo **Slave**.

El bus utiliza principalmente dos líneas de comunicación: **SCL** y **SDA**. La línea `SCL` corresponde al reloj y permite sincronizar la transferencia de información. La línea `SDA` corresponde a los datos y es bidireccional, por lo que puede ser utilizada tanto por el Master como por el Slave.

En la figura también se observan las resistencias *pull-up* conectadas a las líneas `SCL` y `SDA`. Estas resistencias permiten que las líneas permanezcan en nivel lógico alto cuando ningún dispositivo las está llevando a nivel bajo.

El Master es el encargado de iniciar la comunicación, generar el reloj y seleccionar mediante direccionamiento el dispositivo con el que desea comunicarse. La EEPROM responde cuando reconoce su dirección.

<img width="465" height="185" alt="Figura 24-2" src="https://github.com/user-attachments/assets/4f20303b-a71c-4318-ace6-2ecc021f2cd1" />


> “Esta figura muestra la estructura física básica de nuestro sistema I2C. Tenemos un Master conectado a una memoria EEPROM, que actúa como Slave. La comunicación se realiza mediante las líneas SCL y SDA. SCL proporciona el reloj y SDA transporta los datos. Las resistencias pull-up son necesarias porque las líneas del bus se manejan liberando la línea o llevándola a nivel bajo.”

---

### Descripción de la Figura 24-3 – Estados del protocolo I2C

La Figura 24-3 representa las condiciones fundamentales que permiten controlar una comunicación I2C. En ella se pueden identificar principalmente las condiciones de **START**, **transferencia de datos**, **ACK/NACK** y **STOP**.

La condición de **START** se produce cuando `SDA` cambia de nivel alto a nivel bajo mientras `SCL` permanece en nivel alto. Esta transición indica que el Master está iniciando una comunicación.

Durante la transferencia de datos, el valor de `SDA` debe permanecer estable mientras `SCL` está en nivel alto. Los cambios de datos se realizan normalmente mientras `SCL` está en nivel bajo.

Después de transmitir cada byte se utiliza un noveno pulso de reloj para realizar la confirmación. Esta confirmación puede ser un **ACK** o un **NACK**.

Un **ACK** indica que el byte fue recibido correctamente y que la comunicación puede continuar. Un **NACK** indica que el receptor no está confirmando el byte o que el Master ha terminado una recepción.

Finalmente, la condición de **STOP** se genera cuando `SDA` cambia de bajo a alto mientras `SCL` se encuentra en alto. Esta condición indica que la comunicación ha finalizado y que el bus vuelve al estado de reposo.

<img width="575" height="209" alt="Figura 24-3" src="https://github.com/user-attachments/assets/e01ef5dc-c76e-4641-a056-ec591213b34f" />

> “Esta figura representa los estados principales del protocolo I2C. Primero tenemos START, que inicia la comunicación. Después se transmiten los bits de información sincronizados con SCL. Por cada byte existen ocho bits de datos y un noveno ciclo utilizado para ACK o NACK. Finalmente, STOP indica que la comunicación terminó.”

---

### Descripción de la Figura 24-15 – Secuencia de operación del Master I2C

La Figura 24-15 muestra cómo el **Master I2C** controla una operación de comunicación con una memoria EEPROM. La figura relaciona las acciones que realiza el Master con las operaciones propias del controlador I2C.

Entre las operaciones mostradas se encuentran `SEN`, `I2CxTRN`, `RSEN`, `RCEN`, `ACKEN` y `PEN`.

`SEN` corresponde al envío de una condición de **START**.

`I2CxTRN` corresponde a la transmisión de información a través del bus, como la dirección del dispositivo o la dirección interna de memoria.

`RSEN` representa un **Repeated START**, utilizado cuando el Master necesita cambiar de una operación de escritura a una operación de lectura sin liberar el bus.

`RCEN` permite habilitar la recepción de datos desde el dispositivo Slave.

`ACKEN` controla el envío de la confirmación **ACK/NACK** después de una recepción.

`PEN` corresponde a la generación de la condición de **STOP**, finalizando la comunicación.

Esta figura es especialmente importante para el proyecto porque permite relacionar las operaciones del protocolo con los estados que posteriormente se pueden implementar mediante una máquina de estados en la FPGA.

<img width="605" height="424" alt="Figura 24-15" src="https://github.com/user-attachments/assets/b5ab95b2-3da7-43a7-9ffe-d2619e6033bc" />


> “Esta figura muestra cómo el Master controla una comunicación completa con la EEPROM. Primero genera START, después transmite la información necesaria para seleccionar la memoria y su dirección interna. Para una lectura, genera un Repeated START, cambia la operación a lectura y recibe el dato. Finalmente genera ACK o NACK según corresponda y termina con STOP. Esta secuencia nos sirve como referencia para diseñar los estados de nuestro I2C Master en la FPGA.”

---

### Descripción de la Figura 24-4 – Lectura aleatoria de una EEPROM

La Figura 24-4 muestra una operación de **lectura aleatoria** (*Random Read*) de una memoria EEPROM mediante I2C.

La operación comienza con una condición de **START** generada por el Master. Posteriormente se transmite la dirección de la EEPROM acompañada del bit `W`, indicando inicialmente una operación de escritura.

Después de recibir el **ACK** de la EEPROM, el Master envía la dirección interna de memoria que desea leer. La EEPROM confirma nuevamente mediante un ACK.

Una vez enviada la dirección interna, el Master genera un **REPEATED START**. Esto permite iniciar una nueva fase de comunicación sin liberar completamente el control del bus.

Después del REPEATED START, el Master vuelve a enviar la dirección de la EEPROM, pero esta vez con el bit `R`, indicando una operación de lectura.

La EEPROM entonces coloca el dato solicitado sobre la línea `SDA`, mientras el Master genera los pulsos de `SCL` para recibirlo.

Cuando el Master recibe el último dato que necesita, genera un **NACK** para indicar que no desea recibir otro byte. Finalmente, genera la condición de **STOP** y termina la comunicación.

La secuencia completa puede resumirse como:

`START → ADDRESS + W → ACK → MEMORY ADDRESS → ACK → REPEATED START → ADDRESS + R → DATA → NACK → STOP`

<img width="880" height="217" alt="Figura 24-4" src="https://github.com/user-attachments/assets/0801a9b0-a528-404d-9d3d-b046b4429fbb" />

> “Esta figura muestra cómo se realiza una lectura aleatoria de la EEPROM. Primero usamos una operación de escritura para indicarle a la memoria qué posición queremos leer. Después hacemos un Repeated START y volvemos a enviar la dirección, pero ahora con el bit de lectura. La EEPROM entrega el dato por SDA. Cuando ya recibimos el dato que necesitamos, el Master envía un NACK y finalmente genera STOP.”

---

### Relación entre las cuatro figuras

Las cuatro figuras se complementan entre sí y permiten entender el funcionamiento completo del proyecto.

La **Figura 24-2** explica la conexión física del sistema: quién es el Master, quién es el Slave y cuáles son las líneas utilizadas.

La **Figura 24-3** explica las reglas básicas del protocolo: START, transferencia de datos, ACK/NACK y STOP.

La **Figura 24-15** muestra cómo el Master ejecuta las diferentes operaciones necesarias para controlar una comunicación I2C.

La **Figura 24-4** presenta un ejemplo completo de aplicación de esas operaciones: la lectura de una posición específica de una memoria EEPROM.

Por lo tanto, la secuencia lógica para entender las figuras es:

**Conexión física → reglas del protocolo → operaciones del Master → lectura de la EEPROM.**
