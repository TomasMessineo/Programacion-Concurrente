# Teoría de Programación Concurrente | Clase 1 | Introducción

### ¿Qué es la concurrencia?*

Al hablar de concurrencia en general, no nos referimos únicamente a un programa concurrente. 
Básicamente, es la capacidad de ejecutar o realizar múltiples actividades al mismo tiempo o de forma simultanea. En esta materia nos vamos a enfocar en crear un programa concurrente.
Este es un concepto muy importante en el diseño del hardware, para los sistemas operativos (aprovechando la concurrencia del hardware y así permitir utilizar eso en los programas que se creen).
En la vida cotidiana, en todo existe la concurrencia: Desde escuchar, sentir y ver, todo al mismo tiempo.

En algunos casos, se tiende a pensar en sistemas secuenciales en lugar de concurrentes para simplificar el proceso de diseño. Pero esto va en contra de la necesidad de sistemas de cómputo cada vez mas poderosos y flexibles.
Nos va a convenir en muchos casos (lo cual no va a ser muy sencillo ya que hay que cambiar la forma de pensar) pensar la solución a un problema de manera concurrente.

La concurrencia en sí, es un concepto de software no restringido a una arquitectura particular de hardware ni a un número determinado de procesadores. Nos ayuda a evaluar, pensar y distribuir un trabajo entre distintas partes que van ejecutandose de forma simultánea y van, de alguna manera, cooperando y compitiendo por el acceso a recursos.

### ¿Dónde está la concurrencia?

* Navegadores web accediendo una página mientras atiende al usuario
* Varios navegadores accediento a la misma página
* Acceso a disco. Mientras en el SO una aplicación está accediendo al disco para buscar informacion, hay otras apps que pueden estar realizando otras actividades sobre la computadora y el SO en particular
* Los teléfonos, donde uno mientras habla por teléfono puede estar recibiendo otro llamado y mensajes
* Un sistema de reserva de pasajes, donde se tiene en una misma BD que es la que tiene los pasajes que se han vendido o no, puede haber muchas personas que estan accediendo a esa información para poder reservar o comprar sus pasajes. Las personas acceden desde diferentes ubicaciones para comprar el pasaje al mismo tiempo, esto es concurrencia
* Cualquier objeto mas o menos "inteligente", como aquellos sistemas que cuentan con sensores para recolectar información y en base a la misma tomar ciertas decisiones "inteligentes", todos esos sensores o componentes funcionan al mismo tiempo y se estan comunicando o interactuando entre ellos para poder decidir qué es lo que se va a hacer.
* Juegos, automóviles, etc.

---

**Concurrencia "Natural"**

_Problema secuencial con su solución: Desplegar cada 3 segundos un cartel ROJO_

```text
Programa Cartel
	Mientras (true)
		Demorar (3 seg)
		Desplegar cartel
	Fin mientras
Fin Programa
```

_Problema secuencial con su solución: Desplegar cada 3 segundos un cartel ROJO y cada 5 segundos un cartel AZUL:_

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

Las dos variables "Proximo" nos indican cuál es el proximo momento en el cual tiene que ser mostrado el cartel de un color o de  otro. Además, inicio todo en un tiempo 0 (variable "Actual").
En un loop infinito me fijo cual de los 2 es el próximo que tiene que ser mostrado. Me fijo además cuanto tiempo falta para llegar a eso, demoro todo el sistema esa cantidad de tiempo y muestro el cartel que correspondo y actualizo cual es el próximo momento donde tiene que ser mostrado ese cartel.

Si se quieren sumar mas carteles para mostrar dada una determinada cantidad de tiempo, lo que va a convenir es (aprovechando que todo es concurrente) tratar de resolver ese problema directamente de forma CONCURRENTE ya que cada uno de esos carteles es independiente del resto. 

```text
Programa Cartel (color, tiempo)
	Mientras (true)
		Demorar (tiempo segundos)
		Desplegar cartel (color)
	Fin Mientras
Fin Programa
```

Luego, ejecutando este código varias veces con distintos parámetros al mismo tiempo pero con distintos parámetros, estoy desarrollando una solución concurrente. Llamando varias instancias de este código/programa con distintos valores en sus parámetros.

**Una de las cosas que hay que tener en cuenta cuando hablamos de concurrencia es el NO DETERMINISMO.** Esto dice que en ejecuciones concurrentes con la misma entrada (mismos datos de entrada) se pueden obtener soluciones o resultados diferentes, ya que esto va a depender del orden en el que se van ejecutando las cosas. En este caso, supongamos que en un instante x de tiempo coincide que justo hay que mostrar un cartel rojo y un cartel azul... en este caso vamos a tener varias soluciones: una podría ser que muestro uno de los dos (el primero que llega lo imprimo en pantalla y si llega el otro inmediatamente lo sobreescribe y se muestra), o podríamos decir que secuencializamos la muestra (es decir, mostramos el primero que llegó, esperamos un segundo y mostramos el otro por mas de que va a ir atrasado en 2 segundo).
Tenemos el mismo programa concurrente, tenemos las mismas entradas en el mismo, pero por la forma en la que se van ejecutando o intercalando las cosas, podría ser que el resultado que se obtuvo en una de las ejecuciones sea diferente al que se obtuvo en las otras, *esto es el NO DETERMINISMO*

### ¿Por qué es necesaria la Programación Concurrente?

Antes, lo que se hacía si uno quería que una aplicación se ejecute mas rápido era comprarme una máquina mas potente y más rápida, para poder ejecutar la aplicación secuencial de una forma mas rápida.
Si uno tenía los recursos y quería reducir el tiempo de ejecución de un programa, se compraba una máquina mas nueva y con eso lo resolvía.
Al ya no poder seguir mejorando la velocidad de los monoprocesadores (y sus ciclos de reloj), surgen los *multicore* (o ahora los manycore). Si yo quiero reducir el tiempo de ejecución de una aplicación, esta tiene que ser concurrente (no me sirve simplemente con tener multicore o manycore). Es necesario que la app también sea CONCURRENTE para aprovechar al máximo los elementos de procesamiento que forman el multicore o manycore para que trabajen de forma simultanea y así resolver la tarea que tengo que hacer en mi programa reduciendo el tiempo de ejecución. 

También es más facil pensar una solución concurrente para un problema CONCURRENTE en vez de una solución secuencial, ya que se adapta mas al mundo real.

Otra razón por la cual es importante la programación concurrente es para aprovechar los tiempos muertos por ejemplo de hacer una E/S a través de un periférico o de acceder a memoria, y tratar de aprovechar esos momentos para continuar trabajando en esa aplicación. Por ejemplo: Mientras estoy esperando que el usuario me conteste entre la opción A o B, yo podría seguir trabajando o resolviendo otra parte del problema en ese mismo código. De esta forma, para cuando el usuario me responde A o B, yo ya tengo gran parte de lo que tenía que hacer resuelto, sin depender exclusivamente de la opción del usuario. También, cuando se hace acceso a memoria se pueden aprovechar los cores de los procesadores para ir adelantando y ejecutando sentencias que no requieran de esos datos que tuve que ir a buscar a memoria.

En los sistemas distribuidos también se necesita concurrencia. Una característica de estos sistemas es que yo voy a tener en general accesos a un cierto sistema compartido desde distintos lugares, donde esos sistemas habitualmente lo que tienen es la compartición de algún recurso entre sí, por ejemplo, una base de datos, y desde los distintos lugares pueden estar accediendo a través de un sistema distribuido para trabajar con esa misma base de datos (por ejemplo comprar un boleto de avión). También se debe evitar que por ejemplo dos personas compren el mismo pasaje, o el último, a la misma vez.

Por estas razones, hoy en día se ha tornado mas importante la Programación Concurrente.

### Objetivos de los sistemas concurrentes

**Ajustar el modelo** de arquitectura de hardware y software al problema del mundo real a resolver. 
**Incrementar la performance**, mejorando los tiempos de respuesta de los sistemas de cómputo, a través de un enfoque diferente de la arquitectura física y lógica de las soluciones

**Algunas ventajas:**
* Se puede lograr mayor velocidad de ejecución.
* Mejor utilización de la CPU de cada procesador.
* Explotación de la concurrencia que tienen de por sí los problemas reales, sacando provecho de eso haciendo soluciones que en principio sean mas sencillas de pensar.

### Posibles comportamientos de los procesos

**Programa Secuencial:** Un único flujo de control que ejecuta una instrucción y cuando esta finaliza ejecuta la siguiente.
Por ahora, se llamará "Proceso" a un programa **secuencial**. Un proceso es un hilo de ejecución donde las instrucciones se ejecutan una detrás de otra y hasta que no terminó la primera, no se ejecuta la segunda y así sucesivamente.

**Un programa concurrente va a estar formado por varios procesos**. Cada uno de los procesos va a resolver un único problema, no problemas distintos. Yo tengo un único programa que es para resolver un único problema, pero ese programa va a estar compuesto por varios procesos. Esos procesos que forman parte del programa van a tener distintos comportamientos. Pueden ser procesos independientes ya que podrían solucionar su parte sin interactuar con el resto, pero los procesos pueden cooperar o competir entre ellos (estos son dos comportamientos diferentes que a su vez pueden darse al mismo tiempo). 

**Los procesos pueden ser independientes entre ellos, cooperar o competir.**
**Independientes** porque cada uno tiene la capacidad de resolver su parte sin interactuar con el resto. Estos procesos son relativamente raros y poco interesantes.
**Cooperar** porque cada uno ejecuta o realiza una parte de la tarea y después de alguna manera todas esas partecitas tienen que integrarse de alguna manera entre ellas para obtener el resultado final. Este caso no es sencillo ya que hay varias decisiones que hay que tomar. Los procesos deben sincronizarse para poder cooperar correctamente.
**Compiten** porque entre ellos "pelean" por utilizar un recurso compartido (por ejemplo, en el sistema de ventas de pasajes de avion, los usuarios "compiten" por acceder a comprar un pasaje). Este tipo de proceso es típico en Sistemas Operativos y Redes, debido a recursos compartidos. Estos problemas no son tan sencillos de resolver, ya que hay varios problemas que se pueden dar: **Deadlock** (donde ninguno de los procesos va a poder continuar trabajando porque está esperando que el otro proceso libere un cierto recurso compartido, que no lo va a poder liberar porque estará esperando que el primer proceso libere otro recurso, el cual tampoco va a ser liberado. Esto es el "Abrazo Mortal" ya que ninguno de los procesos va a poder continuar su ejecución), Inanición (algunos de los procesos nunca logra acceder al recurso compartido ya que hay otro que le gana. Mientras mas procesos compiten por el recurso, más posibilidades de inanición existen)

Hay momentos donde los procesos donde los procesos serán independientes, luego cooperarán y luego competirán (no es que un proceso nace siendo independiente y muere así, sino que puede variar).

### Diferencias entre procesamiento Secuencial, Concurrente y Paralelo

**Procesamiento Secuencial:**
* Nos fuerza a establecer un estricto orden temporal. Siempre se hace una cosa, hasta que no termine lo primero, no se hace lo segundo, lo tercero hasta que no se termine lo segundo, y así sucesivamente.

**Procesamiento Paralelo:**
* Hay mas de una CPU/máquina trabajando al mismo tiempo para realizar tareas o módulos.
* Es costoso.
* Un problema de este tipo se suele llamar: **Concurrencia con Paralelismo de Hardware**
* *CONSECUENCIAS:*
	1. Menor tiempo para completar una tarea.
	2. Menor esfuerzo individual. 
	3. Paralelismo del hardware.
* *DIFICULTADES:*
	1. Necesidad de compartir recursos evitando conflictos.
	2. Necesidad de esperarse en puntos clave. 
	3. Necesidad de comunicarse.
	4. Tratamiento de las fallas. 
	5. Distribuir el trabajo a realizar de la forma que mejor se adapte dependiendo de la cantidad de CPUs/máquinas con los que se cuente y también la cantidad de tareas a realizar.

**Desarrollo Concurrente**
* Las tareas o módulos a completar se van realizando de a ratos por medio de procesos que van tomando UNA SOLO CPU. En estos casos solo se cuenta con una sola CPU y no varias como en el ejemplo anterior de procesamiento paralelo.
* Cuando hablamos de un problema de este tipo, se suele llamar: **Concurrencia sin Paralelismo de Hardware**. Esto debido a que el procesamiento que se hace SI es concurrente, pero el proceso/máquina sobre el cual trabajo es uno solo. 
* Se reduce el tiempo de procesamiento. No se reduce tanto como en casos de concurrencia con paralelismo de hardware, pero sí que se reduce comparandola con la solución secuencial.
* DIFICULTADES:
	1. Distribución de carga de trabajo. Va a ser menor con respecto al paralelismo, pero aún así debe hacerse.
	2. Necesidad de compartir recursos. Si bien en este caso la CPU es uno solo (esto quiere decir que el recurso que se está utilizando seguro que no lo va a estar queriendo acceder otro proceso), igualmente podría haber alguna regla o variación específica.
	3. Necesidad de esperarse en puntos clave. 
	4. Necesidad de comunicarse.
	5. Necesidad de recuperar el "estado" de cada proceso al retomarlo. Esto es en pocas palabras el "Context Switch" que realiza el sistema operativo para correr distintos procesos en el procesador, y también ir cambiandolos depdendiendo de si el proceso se bloquea o si se debe hacer el context switch. Los procesos pueden estar (como se vió en ISO) en la cola de listos o en la cola de wait (cola de wait = procesos bloqueados, por ejemplo, por una E/S que luego pasarán a la cola de listos)

Un proceso o tarea es un elemento concurrente abstracto que puede ejecutarse simultáneamente con otros procesos o tareas, si el hardware lo permite (por ejemplo los TASKs de ADA).
Un programa concurrente puede tener N procesos habilitados para ejecutarse concurrentemente y un sistema concurrente puede disponer de M procesadores cada uno de los cuales puede ejecutar uno o más procesos.
*Características importantes:*
* Interacción
* No determinismo ---> Dificultades para la interpretación (entender la ejecución de un programa) y debug (debuggear/depurar si encontramos un problema). Hay errores que no son tan sencillos de ver a simple vista, y para solucionarlos hay que afilar conceptos y hacer la práctica.

### Procesos e Hilos

**Procesos:** Cada proceso tiene su propio espacio de direcciones (donde ningún otro proceso puede acceder o modificar esa información) y recursos.

**Procesos livianos, threads o hilos:** 
* Son procesos que tienen su propio Program Counter y su pila de ejecución, pero no controla el "contexto pesado" (por ejemplo, las tablas de página). 
* Cada uno de estos procesos livianos o hilos lo que hace es pertenecer a un proceso, donde el proceso es el que tiene todo el contexto pesado y el hilo lo que tiene es información mas local de lo que necesita para ejecutarse. 
* El espacio de direcciones al cual puede acceder este hilo no es propio, sino que es el mismo que posee el proceso al cual pertenece. Si yo tengo un proceso en el cual generé 25 hilos, los 25 hilos van a poder acceder al espacio de direcciones del proceso al que pertenecen.
* Es menos costoso hacer los context switch entre dos hilos que pertenecen a un mismo proceso, que entre procesos en sí.
* La concurrencia puede estar provista por el lenguaje (Java) o por el Sistema Operativo (C/POSIX)

*Aclaración:* En la materia se le dirán procesos en general a estos dos tipos mencionados para no confundir
