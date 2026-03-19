# Teoría Programación Concurrente - Sincronización por variables compartidas | clase 7

## Locks - Barreras

**Problema de la sección crítica:** Permitir que un pedazo de código se ejecute con exclusión mútua, como una acción atómica de grano grueso.

**Barrera:** Punto de sincronización que todos los procesos deben alcanzar para que cualquier proceso pueda continuar.

En la técnica busy waiting un proceso chequea repetidamente una condición hasta que sea verdadera.

La solución mas trivial es:

```text
process SC[i=i to n]
{
  while (true)
  {
    protocolo de entrada;
    sección crítica
    protocolo de salida
    sección no crítica
  }
}
```

Las soluciones a este problema pueden usarse para implementar sentencias await arbitrarias.

¿Qué propiedades deben satisfacer los protocolos de E/S?
* *Exclusión mutua:* A lo sumo un proceso está en su sección crítica.
* *Ausencia de deadlocks (livelocks):* si 2 o mas procesos tratan de entrar a sus secciones críticas, al menos uno tendrá éxito.
* *Ausencia de demora innecesaria:* Si un proceso trata de entrar a su sección crítica y los otros están en sus secciones no criticas o terminaron, el primero no está impedido de entrar a su sección crítica. No tenemos por qué prohibirle el paso a ese proceso. Un proceso tiene que preguntarle a cada proceso si está en su sección crítica. Si no lo está o terminó, tendría que dejarlo pasar, no bloquearlo ni prohibirle el paso porque sino estoy demorandolo innecesariamente. Por ejemplo, un proceso le dice al proceso 1 que puede avanzar, pero a lo mejor el proceso 1 está haciendo otra cosa y "no quiere" entrar o avanzar... y además el proceso 2 sí quiere y puede pasar o avanzar. Entonces en este caso estaría frenando innecesariamente o demorando al proceso 2 cuando podría acelerar este proceso. Si un proceso 10 le consulta al proceso 1, y el proceso 1 le dice que no quiere pasar, que pase el siguiente, y entonces despues el proceso 10 le pregunta al 2, el 2 no quiere, luego le pregunta al 3 y así sucesivamente, por mas de que se tarden 10 minutos, es una mejor solución ya que no se estaría demorando de forma innecesaria. 
Por ejemplo: Hasta que no pasó en el aula la persona con turno 1, no pasa el 2, ni tampoco el 3, y así sucesivamente. Cuando el 1 termine, incrementa en uno el turno actual y ahí pasa el 2. Si el del turno 2 no quiere pasar o está haciendo otra cosa, sería innecesario o perder tiempo no dejar pasar al 3 si este sí está disponible auque el 2 no esté. 
* *Eventual entrada:* un proceso que intenta entrar a su sección crítica tiene posibilidades de hacerlo (eventualmente lo hará). Si la política es débilmente fair, esta política de eventual entrada se va a cumplir. Se cumple pero no se puede asegurar.

Para una acción atómica condicional <await (B) S;>
SCEnter; while (not B) {SCExit; SCEnter;} S; SCExit;

Si S es skip y B cumple ASV, <await (B);> puede implementarse por medio de:
while(not B) skip;

Ene l ejemplo 2 dado usando await en las diapositivas, satisface las primeras 3 propiedades menos la de eventual entrada, ya que a lo mejor el módulo SC1 lo usa un proceso que da toda la vuelta poniendo in1 en falso, y despues SC2 toma el procesador y justo está en el await esperando a que in1 sea false para entrar a su sección crítica, no va a entrar a su sección crítica ya que va a entontrar in1 en false, y así sucesivamente. Es decir, esta solución está bien pero no cubre este caso especificamente aunque sería un escenario MUY improbable. Por ahora se toma como válido, pero no es la mejor solución.
Ese ejemplo para llevarlo a una solución utilizando N procesos en vez de tener in1 y in2 como variables, tengo una sola general y compartida que puede llamarse "lock".

## Solución de Grano fino: Spin Locks

Objetivo: hacer atómico el await de grano grueso.
Idea: Usar instrucciones como Test y Set (TS), Fetch y Add (FA) o...
TEst y Set devuelve el último valor que tiene Test y Set, pero lo pone en true siempre.

Solución tipo Spin Locks: Los procesos se quedan iterando (spinning) mientras esperan que se limpie lock.

Esta solución cumple las 4 propiedades si el scheduling es fuertemente fair.
Una política débilemnte fair es aceptable (rara vez todos los procesos estan simultáneamente tratando de entrrar a su sección crítica).

## Solución Fair: algoritmo Tie-Breaker

Algoritmo tie-breaker (dos procesos): Protocolo de sección que requiere scheduling solo débilmente fair y no usa instrucciones especiales -> más complejo.
En el ejemplo visto con dos variables (in1 e in2) podía producir deadlock. Pero lo que se hace acá en esta solucion es una variable mas para romper los empates. Usa una variable por cada proceso para indicar que el proceso comenzó a ejecutar su protocolo de entrada a la sección crítica, y una varialble adicional para romper empates, indicando que fue el último en comenzar dicha entrada. Esta ultima variable es compartida y de acceso protegido. 
Este algoritmo se vuelve mucho mas costoso y complejo si hablamos de N procesos

## Solucion fair: Algoritmo ticket

Algoritmo ticket: Se reparten números y se espera a que sea el turno.
Los procesos toman un número mayor que el de cualquier otro que espera ser atendido; luego esperan hasta qeut todos los procesos con número mas chico han sido atendidos
Acá usamos el fetch & Add: FA(v,i), donde se guarda el valor original que entró en la variable, la incrementa segun lo que dice le incremento y devuelve el valor original con el que entró la variable v
En casos límite, se pueden exceder los números utiles al hacer el fetch and add, es raro pero puede pasar y todo se puede romper al sumar

## Solucion fair: Algoritmo Bakery

Esta solución de grano grueso no es implementable directamente:
* La asignación a turno[i] exige calcular el máximo de n valores
* El await referencia una variable compartida dos veces

Esta solución es mas costosa en cuanto a cómputo en comparación con el algoritmo anterior. Además, esta implementación no es directa. Calcular el máximo de un vector y aumentarlo en uno además de hacerlo atómico, es demasiado para el hardware.

Estas ultimas 3 soluciones permiten mantener el orden entre los procesls que hicieron las solicitudes asegurando que un proceso (si los dos intentan entrar a la sección crítica al mismo tiempo) no pueda entrar uno 25 veces y el otro quedarse esperando sin haber entrado ninguna vez.

**Mirar la sincronización por barreras para la clase que viene.**