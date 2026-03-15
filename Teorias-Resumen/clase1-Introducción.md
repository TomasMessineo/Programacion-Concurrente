# Teoría de Programación Concurrente | Clase 1 | Introducción

## ¿Qué es la concurrencia?

Al hablar de concurrencia en general, no nos referimos únicamente a un programa concurrente.
Básicamente, es la capacidad de ejecutar o realizar múltiples actividades al mismo tiempo o de forma simultánea. En esta materia nos vamos a enfocar en crear un programa concurrente.

Este es un concepto muy importante en el diseño del hardware y para los sistemas operativos (aprovechando la concurrencia del hardware para permitir utilizarla en los programas que se creen). En la vida cotidiana existe concurrencia en todo: desde escuchar, sentir y ver, todo al mismo tiempo.

En algunos casos se tiende a pensar en sistemas secuenciales en lugar de concurrentes para simplificar el proceso de diseño, pero esto va en contra de la necesidad de sistemas de cómputo cada vez más poderosos y flexibles. Nos va a convenir, en muchos casos, pensar la solución a un problema de manera concurrente (lo cual no será sencillo, ya que hay que cambiar la forma de pensar).

La concurrencia es un concepto de software no restringido a una arquitectura particular de hardware ni a un número determinado de procesadores. Nos ayuda a evaluar, pensar y distribuir un trabajo entre distintas partes que van ejecutándose de forma simultánea y que, de alguna manera, cooperan y compiten por el acceso a recursos.

## ¿Dónde está la concurrencia?

- Navegadores web accediendo a una página mientras atiende al usuario.
- Varios navegadores accediendo a la misma página.
- Acceso a disco: mientras una aplicación accede al disco para buscar información, otras apps pueden estar realizando otras actividades.
- Los teléfonos, donde uno puede estar hablando mientras recibe otro llamado y mensajes.
- Un sistema de reserva de pasajes, con una misma BD a la que acceden muchas personas al mismo tiempo para reservar o comprar.
- Objetos "inteligentes" con sensores que recolectan información y toman decisiones; todos esos sensores funcionan al mismo tiempo e interactúan entre ellos.
- Juegos, automóviles, etc.

---

## Concurrencia "Natural"

**Problema secuencial:** Desplegar cada 3 segundos un cartel ROJO.
```text
Programa Cartel
    Mientras (true)
        Demorar (3 seg)
        Desplegar cartel
    Fin mientras
Fin Programa
```

**Problema secuencial:** Desplegar cada 3 segundos un cartel ROJO y cada 5 segundos un cartel AZUL.
```text
Programa Cartel
    Proximo_Rojo = 3
    Proximo_Azul = 5
    Actual = 0
    Mientras (true)
        Si (Proximo_Rojo < Proximo_Azul)
            Demorar (Proximo_Rojo - Actual)
            Desplegar cartel ROJO
            Actual = Proximo_Rojo
            Proximo_Rojo = Proximo_Rojo + 3
        sino
            Demorar (Proximo_Azul - Actual)
            Desplegar cartel AZUL
            Actual = Proximo_Azul
            Proximo_Azul = Proximo_Azul + 5
    Fin mientras
Fin Programa
```

Las variables `Proximo_Rojo` y `Proximo_Azul` indican cuál es el próximo momento en que debe mostrarse cada cartel. Se parte de un tiempo 0 (variable `Actual`). En el loop infinito se evalúa cuál es el próximo a mostrar, se demora el sistema esa cantidad de tiempo, se muestra el cartel correspondiente y se actualiza el próximo momento en que debe aparecer.

Si se quieren sumar más carteles con distintos intervalos, conviene resolver el problema directamente de forma **concurrente**, ya que cada cartel es independiente del resto:
```text
Programa Cartel (color, tiempo)
    Mientras (true)
        Demorar (tiempo segundos)
        Desplegar cartel (color)
    Fin Mientras
Fin Programa
```

Ejecutando este código varias veces en paralelo con distintos parámetros se obtiene una solución concurrente.

> **Una de las cosas que hay que tener en cuenta cuando hablamos de concurrencia es el NO DETERMINISMO.** En ejecuciones concurrentes con la misma entrada pueden obtenerse resultados diferentes, ya que esto depende del orden en que se van ejecutando las cosas. Por ejemplo, si en un instante coincide que hay que mostrar un cartel rojo y uno azul, podría mostrarse uno sobreescribiendo al otro, o secuencializarse la muestra. El mismo programa concurrente con las mismas entradas puede producir resultados distintos según cómo se intercalen las ejecuciones: *esto es el NO DETERMINISMO*.

## ¿Por qué es necesaria la Programación Concurrente?

Antes, si se quería que una aplicación se ejecutara más rápido, simplemente se compraba una máquina más potente. Al ya no poder seguir mejorando la velocidad de los monoprocesadores (y sus ciclos de reloj), surgieron los *multicore* (y los *manycore*). Para reducir el tiempo de ejecución de una aplicación en este contexto, la aplicación debe ser **concurrente**: de lo contrario, los múltiples núcleos no se aprovechan.

Además, es más fácil pensar una solución concurrente para un problema naturalmente concurrente, ya que se adapta mejor al mundo real.

Otra razón importante es aprovechar los tiempos muertos de E/S o accesos a memoria. Por ejemplo, mientras se espera la respuesta de un usuario, se puede seguir resolviendo otra parte del problema; cuando el usuario responde, ya se tiene gran parte del trabajo adelantado.

En los **sistemas distribuidos** también se necesita concurrencia. Es habitual que múltiples usuarios accedan a un recurso compartido (como una base de datos) desde distintos lugares al mismo tiempo, por ejemplo para comprar un boleto de avión. Aquí también es necesario evitar que dos personas compren el mismo pasaje simultáneamente.

## Objetivos de los sistemas concurrentes

- **Ajustar el modelo** de arquitectura de hardware y software al problema del mundo real a resolver.
- **Incrementar la performance**, mejorando los tiempos de respuesta mediante un enfoque diferente de la arquitectura física y lógica.

**Algunas ventajas:**
- Mayor velocidad de ejecución.
- Mejor utilización de la CPU de cada procesador.
- Explotación de la concurrencia inherente a los problemas reales, lo que permite soluciones más naturales y sencillas de pensar.

## Posibles comportamientos de los procesos

**Programa Secuencial:** Un único flujo de control que ejecuta una instrucción y, cuando esta finaliza, ejecuta la siguiente. Por ahora, se llamará "Proceso" a un programa secuencial: un hilo de ejecución donde las instrucciones se ejecutan una detrás de otra.

**Un programa concurrente está formado por varios procesos**, cada uno resolviendo una parte del mismo problema. Estos procesos pueden tener distintos comportamientos:

- **Independientes:** cada uno resuelve su parte sin interactuar con el resto. Son relativamente raros y poco interesantes.
- **Cooperantes:** cada uno realiza una parte de la tarea y luego esas partes se integran para obtener el resultado final. Los procesos deben sincronizarse para cooperar correctamente.
- **Competidores:** "pelean" por utilizar un recurso compartido (por ejemplo, usuarios comprando el último pasaje disponible). Este tipo es típico en Sistemas Operativos y Redes. Sus principales problemas son:
  - **Deadlock ("Abrazo Mortal"):** ningún proceso puede continuar porque cada uno espera que el otro libere un recurso que tampoco liberará.
  - **Inanición:** algún proceso nunca logra acceder al recurso compartido porque otro siempre le gana. Cuantos más procesos compiten, mayor es la posibilidad de inanición.

Un proceso no nace con un comportamiento fijo: puede comenzar siendo independiente, luego cooperar y luego competir.

## Diferencias entre procesamiento Secuencial, Concurrente y Paralelo

### Procesamiento Secuencial
Impone un estricto orden temporal: siempre se hace una cosa a la vez, y hasta que no termina lo primero no se comienza lo segundo.

### Procesamiento Paralelo
- Más de una CPU/máquina trabajando simultáneamente.
- También llamado **Concurrencia con Paralelismo de Hardware**.
- Es costoso.

**Consecuencias:**
1. Menor tiempo para completar una tarea.
2. Menor esfuerzo individual.
3. Aprovechamiento del paralelismo del hardware.

**Dificultades:**
1. Necesidad de compartir recursos evitando conflictos.
2. Necesidad de sincronización en puntos clave.
3. Necesidad de comunicación entre procesos.
4. Tratamiento de fallas.
5. Distribución del trabajo según la cantidad de CPUs y tareas disponibles.

### Desarrollo Concurrente (sin paralelismo de hardware)
- Las tareas se realizan de a ratos mediante procesos que comparten **una sola CPU**.
- También llamado **Concurrencia sin Paralelismo de Hardware**.
- Se reduce el tiempo de procesamiento, aunque no tanto como con paralelismo de hardware.

**Dificultades:**
1. Distribución de carga de trabajo (menor que en el paralelo, pero igualmente necesaria).
2. Necesidad de compartir recursos.
3. Necesidad de sincronización en puntos clave.
4. Necesidad de comunicación entre procesos.
5. Necesidad de recuperar el "estado" de cada proceso al retomarlo (*Context Switch*): los procesos pueden estar en la cola de listos o en la cola de espera (bloqueados, por ejemplo por una E/S).

**Características importantes de la concurrencia:**
- Interacción entre procesos.
- No determinismo → dificultades para interpretar la ejecución y para depurar errores, que a veces no son evidentes a simple vista.

## Procesos e Hilos

**Procesos:** cada proceso tiene su propio espacio de direcciones (inaccesible para otros procesos) y sus propios recursos.

**Procesos livianos, threads o hilos:**
- Tienen su propio Program Counter y pila de ejecución, pero no controlan el "contexto pesado" (como las tablas de página).
- Pertenecen a un proceso, que es quien posee el contexto pesado; el hilo solo mantiene la información local necesaria para su ejecución.
- Comparten el espacio de direcciones del proceso al que pertenecen: si un proceso tiene 25 hilos, los 25 acceden al mismo espacio de direcciones.
- El context switch entre hilos de un mismo proceso es menos costoso que entre procesos distintos.
- La concurrencia puede estar provista por el lenguaje (Java) o por el Sistema Operativo (C/POSIX).

> **Aclaración:** en la materia se usará el término "proceso" para referirse a ambos tipos, para no generar confusión.
