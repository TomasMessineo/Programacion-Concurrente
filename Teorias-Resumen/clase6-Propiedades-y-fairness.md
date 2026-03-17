# Teoría de Programación Concurrente | Clase 6 | Propiedades y Fairness

## Propiedades de seguridad y vida

Una propiedad en un programa concurrente es un atributo o característica que es verdadera en cualquiera de las posibles historias que se puedan llegar a dar. 

Todas las propiedades de los programas concurrentes se pueden clasificar en 2 tipos:
**Propiedades de seguridad (safety):**
* Nada malo le ocurre a un proceso: Asegura estados consistentes 
* Una falla de seguridad indica que algo anda mal (que hay un error en el programa).
* Ejemplos: Exclusión mutua, ausencia de interferencia entre procesos, partial correctness (propiedad que garantiza que si el programa termina --puede ser que NO lo haga--, el resultado será el correcto, es decir, no habrá errores en la salida final).
**Propiedades de vida (liveness):**
* Eventualmente ocurre algo bueno con una actividad: progresa, no hay deadlocks.
* Una falla de vida indica que las cosas dejan de ejecutar.
* Ejemplos: Terminación (propiedad que nos asegura que el programa va a terminar en cualquiera de las posibles historias del mismo. Esta propiedad no asegura que termine bien o mal el programa, pero sí asegura que en algún momento va a terminar y no se va a bloquear o colgar en el medio), asegurar que un pedido de servicio será atendido, que un mensaje llega a destino, que un proceso eventualmente alcanzará su SC, ausencia de inanición, ausencia de deadlock, etc -> Dependen de las políticas de scheduling

**Total correctness:** La corrección total es aquella propidad la cual asegura que el programa concurrente siempre va a terminar pero también asegura que este programa siempre termina BIEN. Esta es Corrección Parcial + Terminación.

## Fairness y políticas de scheduling

El fairness lo que trata de hacer es garantizar que los procesos tengan chance de avanzar, sin importar lo que hagan los demás.
Una acción atómica en un proceso es elegible si es la próxima acción atómica que debe ejecutarse en ese proceso. Si tenemos varios procesos que forman parte de un programa concurrente, vamos a tener varias acciones atómicas elegibles (una por cada proceso). 
Una **política de scheduling** determina cuál de esas acciones atómicas elegibles es la próxima que va a ejecutarse en el programa concurrente.

**Fairness Incondicional:** Una política de scheduling es incondicionalmente fair si toda acción atómica incondicional que es elegible eventualmente es ejecutada. En el tiempo anterior, Round Robin es incondicionalmente fair en monoprocesador, y la ejecución paralela lo es en un multiprocesador.
**Fairness Débil:** Una política de scheduling es débilmente fair si:
1. Es incondicionalmente fair y
2. Toda acción atómica que se vuelve elegible eventualmente es algún momento va a ser ejecutada si esa condición se vuelve true en algún momento, y esta permanece true hasta que es vista por el proceso que ejecuta la acción atómica condicional.

No es suficiente para asegurar que cualquier sentencia await elegible eventualmente se ejecuta: la guarda podría cambiar el valor (de false a true y nuevamente a false) mientras un proceso está demorado.

**Fairness Fuerte:** Una política de scheduling es fuertementa fair si:
1. Es incondicionalmente fair y
2. Toda acción atómica condicional que se vuelve elegible eventualmente es ejecutada pues su guarda se convierte en true con infinita frecuencia.
