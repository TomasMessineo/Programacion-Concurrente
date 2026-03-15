# Teoría de Programación Concurrente | Clase 3 | Concurrencia a nivel de Hardware

## Niveles de memoria

En las máquinas de memoria compartida (como los Multicore), los niveles de memoria se organizan de la siguiente manera:
```
Memoria Principal
       |
 Caché nivel 2
       |
 Caché nivel 1
       |
      CPU
```

Cuanto más cerca se está de los registros de la CPU, menor es el tamaño de esa memoria pero mayor es la velocidad de acceso. A medida que nos alejamos hacia la memoria principal, el tamaño aumenta pero la velocidad de acceso disminuye notablemente. Se respetan los principios de localidad temporal y espacial de referencia.

### Coherencia de caché

¿Qué pasa si diferentes procesos que pueden estar en diferentes unidades de procesamiento modifican una variable compartida? En estos casos, las máquinas de memoria compartida cuentan con un **protocolo de coherencia de caché** para mantener la consistencia: garantiza que cuando un proceso modifica un dato compartido que podría estar replicado en más de una caché, todos los demás procesos trabajen con el valor actualizado. En un monoprocesador este problema no existe, ya que hay una sola memoria principal y una sola jerarquía de caché.

## Máquinas de Memoria Compartida vs. Memoria Distribuida

### Memoria Compartida

En estos sistemas, múltiples procesadores comparten una misma memoria principal. Existen dos esquemas principales:

- **UMA (Uniform Memory Access):** varias CPUs, cada una con su caché, conectadas todas a una única memoria general. El tiempo de acceso a memoria es el mismo para todos los procesadores. La comunicación entre procesos se puede hacer directamente a través de esa memoria compartida.

- **NUMA (Non-Uniform Memory Access):** cada CPU tiene su propia caché y su propio banco de memoria local, pero también puede acceder a la memoria de otras CPUs a través de una interconexión. El tiempo de acceso varía según si el dato está en la memoria local o en la de otro procesador (de ahí el "non-uniform"). Se usa para escalar a mayor cantidad de procesadores donde un bus compartido sería un cuello de botella.

Un ejemplo concreto de memoria compartida es un **multicore de 8 núcleos**: todos los núcleos comparten la memoria principal y se comunican a través de ella. Las **GPUs** también son un ejemplo, con cientos o miles de núcleos que comparten memoria dentro del mismo chip.

### Memoria Distribuida

En estos sistemas, varios procesadores están conectados a través de una **red de interconexión** (que puede ir desde una red local hasta Internet). Cada máquina tiene su propia memoria local y no puede acceder directamente a la memoria de otra.

En la práctica, cada nodo de este sistema suele ser en sí mismo un multicore, por lo que dentro de cada nodo existe memoria compartida entre sus núcleos. Esto da lugar a dos enfoques de comunicación:

- **Solo pasaje de mensajes:** se trata a todo el sistema como distribuido, ignorando la memoria compartida dentro de cada nodo. Es más simple de razonar pero no aprovecha al máximo el hardware.
- **Programación híbrida:** se usa pasaje de mensajes para la comunicación *entre nodos* y memoria compartida para la comunicación *dentro de cada nodo*. Es más complejo pero más eficiente.

En resumen: la diferencia clave es que en memoria compartida todos los procesadores "ven" la misma memoria y pueden comunicarse leyendo y escribiendo en ella, mientras que en memoria distribuida cada procesador solo ve su memoria local y la comunicación con los demás debe hacerse explícitamente mediante el envío de mensajes.

## Evolución histórica de la concurrencia

| Época | Hitos |
|---|---|
| 60's | Evolución de los SO. Más procesadores por chip para mayor potencia de cómputo. |
| 70's | Formalización de la concurrencia en los lenguajes de programación. |
| 80's | Redes y procesamiento distribuido. |
| 90's | MPP, Internet, Client/Server, Web computing. |
| 2000's | SDTR, computación móvil, cluster y multicluster, sistemas colaborativos, computación pervasiva y ubicua, grid computing, virtualización. |
| Hoy | Big Data, IA, computación elástica, cloud computing, green computing, bioinformática, redes de sensores, IoT, banca electrónica. |
