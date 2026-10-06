## **Diagrama de bloques general:** 
<img width="736" height="477" alt="Captura desde 2026-10-05 05-35-00" src="https://github.com/user-attachments/assets/8f54781b-395f-4de5-840c-a57e91b409fa" />

**Figura 1. Diagrama de bloques general protocolo I2C.**

Example of Single-Master Transmit/Receive Using I2C Bus Interface. http://www.renesas.com/. abril 2010

* **MASTER:** Decide qué operación I²C realizar.
* <img width="412" height="651" alt="Diagrama sin título(1)" src="https://github.com/user-attachments/assets/0a72a99a-ca79-475a-afae-48b39216bdbe" />

**Figura 1.2. Diagrama de bloques MASTER protocolo I2C.**

* **CONTROL UNIT:** Decide qué operación debe realizarse y en qué momento, mediante una máquina de estados (FSM).
<img width="412" height="651" alt="CONTROLBLOCK" src="https://github.com/user-attachments/assets/8fa5d96a-8d2f-49b9-836d-65e02f78a2f3" />

**Figura 1.2.1 Diagrama de bloques CONTROL UNIT protocolo I2C.**

* **DATA PATH:** Contiene los registros, contadores, multiplexores y demás circuitos que manipulan, transmiten y reciben los datos durante la comunicación.
<img width="412" height="561" alt="pathBLOCK" src="https://github.com/user-attachments/assets/69c5b2e3-d7c3-4dc4-bfd5-da1c5bdddc2e" />

**Figura 1.3 Diagrama de bloques DATA PATH protocolo I2C.**

* **MASTER FINAL:** Controla y coordina la comunicación I²C con el dispositivo esclavo, generando las señales SCL y SDA necesarias para leer o escribir datos.
* <img width="528" height="1062" alt="GENERALBLOCK" src="https://github.com/user-attachments/assets/b1c6b2d0-25a4-491f-b3ad-0cb23963d355" />
**Figura 2 Diagrama de bloques I2C MASTER diseño protocolo I2C.**
