# Teoría de Programación Concurrente | Clase 2 | Conceptos básicos de concurrencia

## Comunicación entre procesos

Una de las características de los programas concurrentes es que en algún momento un proceso puede necesitar un dato generado por otro proceso para poder avanzar en la solución del problema. Ante esto surge el concepto de **comunicación entre procesos concurrentes**: el modo en que se organizan y transmiten los datos entre tareas concurrentes, mediante protocolos que controlan cómo progresa esa comunicación y verifican que se haya realizado correctamente.

### Tipos de comunicación

La comunicación entre procesos concurrentes se puede identificar en dos grandes grupos:

- **Comunicación por Memoria Compartida:** se necesita un lugar de memoria al que todos los procesos puedan acceder. Para comunicarse, los procesos depositan datos en esas posiciones de memoria y otros los leen desde allí. Como no pueden operar simultáneamente sobre la memoria compartida, es obligatorio bloquear y liberar el acceso.
- **Comunicación por Pasaje de Mensajes:** las unidades de procesamiento se encuentran en máquinas separadas sin memoria compartida, por lo que la comunicación se realiza enviando mensajes a través de una red de interconexión. Es necesario establecer un canal físico (como un cable) o lógico (simulado en memoria) para transmitir información. Para que la comunicación sea efectiva, los procesos deben saber cuándo tienen mensajes para leer y cuándo deben transmitir.

## Sincronización

Es la posesión de información acerca de otro proceso para coordinar actividades. Los procesos se sincronizan para poder avanzar. La sincronización y la comunicación son los dos componentes fundamentales de los programas concurrentes.

Los procesos se pueden sincronizar de dos maneras:

- **Por exclusión mutua:** asegura que un único proceso a la vez acceda a un recurso compartido que solo puede ser usado por uno a la vez (por ejemplo, una variable compartida o una impresora).
- **Por condición:** bloquea la ejecución de un proceso hasta que se cumpla una condición dada.

## Interferencia

Un proceso toma una acción que invalida las suposiciones hechas por otro proceso. Por ejemplo, un proceso lee el estado actual del programa (variables compartidas) y cuando va a trabajar en base a ese estado, otro proceso ya lo modificó, lo cual puede generar un error.

## Prioridad

Cuando un proceso tiene mayor prioridad que otro, puede apropiarse del procesador o de un recurso compartido, causando que el proceso de menor prioridad sea suspendido.

## Granularidad

La granularidad de una aplicación está dada por la relación entre el cómputo y la comunicación:

- **Grano fino:** poco cómputo seguido de comunicación frecuente. Conviene cuando se tienen muchas unidades de procesamiento poco potentes.
- **Grano grueso:** grandes bloques de cómputo seguidos de comunicación. Conviene en arquitecturas de tipo cluster o distribuidas.

## Manejo de recursos

Uno de los temas principales de la programación concurrente es la administración de recursos compartidos, lo cual incluye su asignación, métodos de acceso, bloqueo y liberación, seguridad y consistencia.

Una propiedad deseable es el **equilibrio en el acceso** (*fairness*): todos los procesos deben tener posibilidades similares de acceder a los recursos compartidos; no debería ocurrir que un proceso acceda 25 veces mientras otro no accedió ninguna.

Dos situaciones **no deseadas** son:
- **Inanición:** un proceso no logra acceder a los recursos compartidos.
- **Overloading:** la carga asignada a un proceso excede su capacidad de procesamiento.

### Deadlock

Otro problema importante a evitar es el **deadlock**. Su ausencia es una propiedad necesaria en los sistemas concurrentes. Las 4 condiciones necesarias y suficientes para que exista son:

1. **Recursos reusables serialmente:** los recursos compartidos solo pueden ser utilizados por un proceso a la vez, con exclusión mutua.
2. **Adquisición incremental:** los procesos adquieren los recursos de a uno y los retienen; no esperan a que todos estén libres para tomarlos juntos.
3. **No apropiación (no-preemption):** ningún proceso puede quitarle un recurso a otro; solo el propio proceso lo liberará voluntariamente al terminar de usarlo.
4. **Espera cíclica:** existe una cadena circular de procesos donde cada uno retiene un recurso que el siguiente en el ciclo necesita, por lo que ninguno puede avanzar.

> Para evitar el deadlock hay que asegurarse de que al menos una de estas 4 condiciones **no pueda darse**.

## Requerimientos para un lenguaje concurrente

Independientemente del mecanismo de comunicación o sincronización requerido, los lenguajes de programación deben proveer librerías o herramientas adecuadas para la especificación e implementación de los mismos.
