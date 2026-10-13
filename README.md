

#  Interfaz de Mouse PS/2

##  Descripción

Este proyecto implementa la comunicación entre un mouse y un sistema anfitrión utilizando el protocolo **PS/2**.

El objetivo es utilizar un mouse PS/2 como periférico de entrada para una consola de videojuegos, interpretando los datos enviados por el mouse para obtener información sobre el movimiento y el estado de sus botones.

La comunicación PS/2 utiliza dos señales principales:

- **Clock:** señal de reloj.
- **Data:** transmisión de datos.

```text
Mouse                  Sistema
  │                      │
  ├────── CLOCK ────────►
  │                      │
  ├────── DATA ─────────►
  │                      │
  └──────────────────────┘
```

El protocolo permite comunicación bidireccional, por lo que el sistema anfitrión también puede enviar comandos al mouse.

---

##  Funcionamiento en Mouse

El mouse PS/2 utiliza una comunicación serial síncrona entre el dispositivo y el sistema anfitrión.

Cuando el ratón PS2 envía información, debe enviar tres paquetes de datos consecutivos. Cada paquete contiene información diferente sobre el botón pulsado, el movimiento y la dirección del mismo.

El mouse a utilizar proporciona información sobre:

<table>
  <thead>
    <tr>
      <th></th>
      <th>Bit 7</th>
      <th>Bit 6</th>
      <th>Bit 5</th>
      <th>Bit 4</th>
      <th>Bit 3</th>
      <th>Bit 2</th>
      <th>Bit 1</th>
      <th>Bit 0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Byte 1</b></td>
      <td>Y overflow</td>
      <td>X overflow</td>
      <td>Y sign bit</td>
      <td>X sign bit</td>
      <td>Always 1</td>
      <td>Middle Btn</td>
      <td>Right Btn</td>
      <td>Left Btn</td>
    </tr>
    <tr>
      <td><b>Byte 2</b></td>
      <td colspan="8" align="center">X Movement</td>
    </tr>
    <tr>
      <td><b>Byte 3</b></td>
      <td colspan="8" align="center">Y Movement</td>
    </tr>
    <tr>
      <td><b>Byte 4</b></td>
      <td colspan="8" align="center">Z Movement</td>
    </tr>
  </tbody>
</table>

Al encenderse, el mouse realiza un proceso de autodiagnóstico denominado **BAT (Basic Assurance Test)**.

Si el diagnóstico es exitoso, el mouse envía un código de confirmacion:

```text
0xAA
```

Posteriormente envía su identificador:

```text
0x000
```

El identificador `0x00` corresponde al mouse PS/2 estándar.

---

## Conexiones fisicas del dispositivo

El ps/2 del mouse consta de un conector de 5 o 6 pines (dependiendo del dispositivo), los cuales asignan 4 pines en ambos casos, y es posible usar un adaptador de uno a otro, o lograr usar algún pin para otro proposito.

<img src="imagenes/conexiones1.png" width="300" alt="Conexiones del dispositivo ps/2">

<img src="imagenes/conexiones2.png" width="300" alt="Conexiones del dispositivo ps/2">



## Forma de onda

<img src="imagenes/señales.png" width="500" alt="Especificaciones de señal del ps/2">



##  Paquete de datos

Un mouse PS/2 estándar transmite la información de movimiento mediante paquetes de **4 bytes**.

### Byte 1

```text
Bit 7    Y Overflow
Bit 6    X Overflow
Bit 5    Y Sign
Bit 4    X Sign
Bit 3    Siempre 1
Bit 2    Botón central
Bit 1    Botón derecho
Bit 0    Botón izquierdo
```

### Byte 2

```text
Movimiento en X
```

### Byte 3

```text
Movimiento en Y
```

### Byte 4

```text
Movimiento en Z
```

---

##  Interpretación de los datos

El primer byte permite determinar el estado de los botones y el signo del movimiento.

Los movimientos X e Y se representan mediante valores de **9 bits en complemento a dos**.

El rango de movimiento representable es:

```text
-255 a +255
```
El cuarto byte representa el movimiento de la rueda mediante un valor de **4 bits en complemento a dos**.

```text
Bit 7 → Extensión de signo
Bit 6 → Extensión de signo
Bit 5 → Extensión de signo
Bit 4 → Extensión de signo
Bit 3 → Z3
Bit 2 → Z2
Bit 1 → Z1
Bit 0 → Z0
```
El rango es:

```text
-8 a +7
```



---

---


##  Modos de operación

El protocolo PS/2 contempla diferentes modos de funcionamiento:

### Reset Mode

Se utiliza durante el encendido del mouse o cuando recibe el comando:

```text
0xFF
```

El mouse realiza el diagnóstico BAT y restablece sus parámetros.

### Stream Mode

Es el modo utilizado normalmente.

El mouse transmite información cuando detecta:

- Movimiento.
- Cambio en el estado de un botón.

### Wrap Mode

Es un modo utilizado principalmente para comprobar la comunicación entre el mouse y el sistema anfitrión.

---

##  Comandos principales

| Comando | Función |
|--------:|---------|
| `0xFF` | Reset |
| `0xFE` | Resend |
| `0xF6` | Restaurar valores predeterminados |
| `0xF5` | Deshabilitar transmisión de datos |
| `0xF4` | Habilitar transmisión de datos |
| `0xF3` | Configurar frecuencia de muestreo |
| `0xF2` | Obtener ID del dispositivo |
| `0xF0` | Activar Remote Mode |
| `0xEE` | Activar Wrap Mode |
| `0xEC` | Salir de Wrap Mode |
| `0xE8` | Configurar resolución |
| `0xE7` | Escalamiento 2:1 |
| `0xE6` | Escalamiento 1:1 |

---


## Microsoft IntelliMouse

Para este proyecto se utilizará principalmente el modo **Microsoft IntelliMouse**, que permite incorporar una rueda de desplazamiento al protocolo PS/2 estándar.

A diferencia del mouse PS/2 estándar, que utiliza paquetes de 3 bytes, el modo IntelliMouse utiliza paquetes de **4 bytes**.

###  Activación del modo IntelliMouse

Para activar el modo IntelliMouse se utiliza la siguiente secuencia:

```text
F3 C8
F3 64
F3 50
F2
```

Donde:

- `F3 C8` → Configura una tasa de muestreo de 200 informes/s.
- `F3 64` → Configura una tasa de muestreo de 100 informes/s.
- `F3 50` → Configura una tasa de muestreo de 80 informes/s.
- `F2` → Solicita el identificador del mouse.

Si el mouse es compatible con IntelliMouse, responderá:

```text
03
```

El identificador `0x03` corresponde al modo IntelliMouse con rueda.


### Byte 4 — Movimiento de la rueda


El sistema utilizará:

- Movimiento en X.
- Movimiento en Y.
- Botón izquierdo.
- Botón derecho.
- Botón central.
- Movimiento de la rueda.

---

##  Aplicación en el proyecto

En este proyecto, el sistema debe:

1. Inicializar la interfaz PS/2.
2. Esperar la respuesta de inicialización del mouse.
3. Detectar el paquete de datos.
4. Recibir los tres bytes del paquete.
5. Verificar los bits de control.
6. Extraer los valores de X y Y.
7. Detectar el estado de los botones.
8. Utilizar estos datos como entradas para la consola.



---


## Secuencia de inicialización del mouse PS/2

```mermaid
sequenceDiagram
    participant H as Host
    participant M as Mouse

    H->>M: FF - Reset
    M-->>H: FA - ACK
    M-->>H: AA - Self-test OK
    M-->>H: 00 - ID inicial

    H->>M: F3 - Configurar muestreo
    H->>M: C8, 64, 50 - Secuencia para rueda
    M-->>H: FA - ACK después de cada dato

    H->>M: F2 - Consultar ID
    M-->>H: FA, 03 - Mouse con rueda

    H->>M: F4 - Enable data reporting
    M-->>H: FA - ACK

    M-->>H: Paquete de 4 bytes
```

### Descripción de la secuencia

- **FF – Reset:** reinicia el mouse.
- **FA – ACK:** confirma la recepción y aceptación de un comando.
- **AA – Self-test OK:** indica que el autotest terminó correctamente.
- **00 – ID inicial:** identifica el dispositivo antes de configurar el modo de rueda.
- **F3 – Configurar muestreo:** permite enviar la secuencia especial de velocidades `200, 100, 80` para activar el modo IntelliMouse.
- **F2 – Consultar ID:** permite verificar el modo de funcionamiento; `03` identifica el modo con rueda.
- **F4 – Enable data reporting:** habilita el envío automático de datos.
- **Paquete de 4 bytes:** contiene el estado de los tres botones, los desplazamientos X/Y y el movimiento de la rueda.


##  Referencias

- Adam Chapweske, **PS/2 Mouse Interfacing**  
  https://isdaman.com/alsos/hardware/mouse/ps2interface.htm

- PS/2 Mouse Protocol  
  https://wiki.osdev.org/PS/2_Mouse

---

##  Integrantes

1. Daniel Gustavo Arango Yela
2. Juan Sebastián Rincon Aguilera
3. .

---
