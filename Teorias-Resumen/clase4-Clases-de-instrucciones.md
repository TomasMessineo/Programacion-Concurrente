# Teoría de Programación Concurrente | Clase 4 | Clases de Instrucciones

Como se mencionó en clases anteriores, un programa concurrente está formado por un conjunto de programas secuenciales.
La programación secuencial estructurada puede expresarse con 3 clases de instrucciones básicas: asignación, alternativa (decisión, como el `if`) e iteración (repetición con condición, como `for` o `while`).
Se requiere una clase de instrucción adicional para representar la concurrencia.

## Asignación

La sintaxis que se va a manejar en esta materia es al estilo C:

- **Asignación simple:** `x = e`
- **Sentencia de asignación completa:** `x = x + 1; y = y - 1; z = x + y;` / `a[3] = 6; aa[2,5] = a[4]`
- **Llamado a funciones:** `x = f(y) + g(6) - 7`
- **Swap:** `v1 :=: v2`
- **Skip:** termina inmediatamente y no tiene efecto sobre ninguna variable del programa. Se utiliza para completar estructuras de control sin hacer nada.

## Estructuras de control

### Alternativa

**Sentencia de alternativa simple:**
```text
if B -> S
fi
```
`B` es una expresión booleana y `S` una instrucción simple o compuesta. `B` "guarda" a `S`, pues `S` no se ejecuta si `B` no es verdadera. El `if` evalúa todas las condiciones booleanas y, de aquellas que son verdaderas, elige una de forma no determinística y ejecuta el conjunto de sentencias asociado. Si todas son falsas, sale sin hacer nada.

**Sentencia de alternativa múltiple:**
```text
if B1 -> S1
□ B2 -> S2
...
□ Bn -> Sn
fi
```
Las guardas se evalúan en algún orden arbitrario. La elección es no determinística: si más de una guarda es verdadera, se elige una al azar. Si ninguna guarda es verdadera, el `if` no tiene efecto.

**Otra sintaxis equivalente:**
```text
if (cond) S;
if (cond) S1 else S2;
```

---

### Iteración

**Sentencia de alternativa iterativa múltiple:**
```text
do B1 -> S1
□ B2 -> S2
...
□ Bn -> Sn
od
```
Las sentencias guardadas se evalúan y ejecutan hasta que **todas** las guardas sean falsas. La elección es no determinística si más de una guarda es verdadera. A diferencia del `if`, al terminar de ejecutar una opción vuelve a evaluar todas las condiciones y repite hasta que ninguna sea verdadera.

**Ejemplos:**
```text
Ejemplo 1:                        Ejemplo 2:
do p > 0 → p = p - 2             do p > 2 → p = p * 2
□ p < 0 → p = p + 3              □ p < 2 → p = p * 3
□ p == 0 → p = random(x)         od
od
```
```text
Ejemplo 3:                        Ejemplo 4:
do p > 0 → p = p - 2             do p == 1 → p = p * 2
□ p > 3 → p = p + 3              □ p == 2 → p = p + 3
□ p > 6 → p = p / 2              □ p == 4 → p = p / 2
od                                od
¿Qué sucede con p = 0, 3, 6, 9?
```

**For-all:** forma general de repetición e iteración.
```text
fa cuantificadores → sentencias af
```
El cuerpo del `fa` se ejecuta una vez por cada combinación de valores de las variables de iteración. Si hay cláusula `st` (*such-that*), la iteración solo se ejecuta para las combinaciones donde `B` es verdadera.

**Ejemplos:**
```text
fa i := 1 to n → a[i] = 0 af
// Inicialización de un vector

fa i := 1 to n, j := i+1 to n → m[i,j] :=: m[j,i] af
// Trasposición de una matriz

fa i := 1 to n, j := i+1 to n st a[i] > a[j] → a[i] :=: a[j] af
// Ordenación de menor a mayor de un vector
```

**Otra sintaxis equivalente:**
```text
while (cond) S;
for [i = 1 to n, j = 1 to n st (j mod 2 = 0)] S;
```

---

### Concurrencia

**Sentencia `co`:**
```text
co S1 // ... // Sn oc
```
Ejecuta las `Si` tareas concurrentemente. La ejecución del `co` termina cuando **todas** las tareas terminaron. Se usa cuando cada hilo va a ejecutar un código diferente.

Con cuantificadores:
```text
co [i=1 to n] { a[i]=0; b[i]=0 } oc
```
Crea `n` tareas concurrentes. Cada hilo ejecuta el mismo código pero con un valor distinto de `i` (el primero con `i=1`, el segundo con `i=2`, etc.).

---

**`Process`:** otra forma de representar concurrencia. Los procesos se definen de antemano y comienzan a correr todos juntos desde el inicio del programa, terminando cada uno cuando agota su código. Todos tienen la misma prioridad.
```text
process A { sentencias }        // proceso único independiente
```

Con cuantificadores:
```text
process B [i=1 to n] { sentencias }   // n procesos independientes
```
Se generan `n` copias del proceso `B`, cada una con un valor distinto de `i`.

**Ejemplo — ¿qué imprime en cada caso? ¿son equivalentes?**
```text
process imprime10 {
    for [i=1 to 10] write(i);
}

process imprime1 [i= 1..10] {
    write(i);
}
```
> **No determinismo...** El primer proceso imprime los números del 1 al 10 en orden secuencial. El segundo crea 10 procesos concurrentes, cada uno imprimiendo su valor de `i`, por lo que el orden de salida es no determinístico y puede variar en cada ejecución. **No son equivalentes.**

---

**Diferencia clave entre `co` y `process`:** `process` ejecuta en *background*, mientras que el código que contiene un `co` espera a que todos los procesos creados terminen antes de continuar con la siguiente sentencia.
