# Teoría de Programación Concurrente | Clase 5 | Acciones Atómicas y Sincronización

## Atomicidad de grano fino

### Estado de un programa concurrente

El **estado** de un programa concurrente es el conjunto de valores que tienen en un momento dado todas las variables: las compartidas entre procesos y las locales de cada proceso.

Cuando un programa concurrente se ejecuta, cada proceso va ejecutando sus sentencias una a una, y cada sentencia transforma ese estado. Una **acción atómica** es una transformación de estado **indivisible**: ningún otro proceso puede ver los estados intermedios de esa acción, simplemente ve el estado anterior y el estado posterior.

### Historias

La secuencia de estados por los que pasa un programa durante su ejecución se llama **historia**. En un programa concurrente, los procesos van intercalando sus acciones atómicas, y esa intercalación define qué historia se obtiene. No todas las historias posibles son válidas: la sincronización permite restringir cuáles se permiten.

> **La sincronización por condición permite restringir las historias de un programa concurrente para asegurar el orden temporal necesario.**

**Ejemplo — historias válidas e inválidas:**
```text
int buffer;

process 1             process 2
{ int x;              { int y;
  while (true)          while (true)
    p1.1: read(x);        p2.1: y = buffer;
    p1.2: buffer = x; }   p2.2: print(y);  }

Posibles historias:
✔ p11, p12, p21, p22, p11, p12, p21, p22, ...
✔ p11, p12, p21, p11, p22, p12, p21, p22, ...
✗ p11, p21, p12, p22, ....
✗ p21, p11, p12, ....
```
Las historias inválidas son aquellas donde `p2.1` lee el buffer antes de que `p1.2` haya escrito en él. Se debe asegurar un orden temporal entre las acciones → las tareas se intercalan → deben fijarse restricciones.

---

### ¿Qué es una acción atómica de grano fino?

Las acciones atómicas de **grano fino** son las que implementa directamente el hardware: operaciones muy básicas como leer un valor de memoria o escribirlo. Son indivisibles a nivel de hardware.

Lo importante es entender que operaciones que parecen simples en el código **no son atómicas** a nivel de hardware. Por ejemplo:

**¿La asignación `A = B` es atómica?**
```text
NO → se descompone en:
  (i)  Load PosMemB, reg     // carga B en un registro
  (ii) Store reg, PosMemA    // guarda el registro en A
```

**¿Y `X = X + X`?**
```text
Se descompone en:
  (i)   Load PosMemX, Acumulador
  (ii)  Add PosMemX, Acumulador
  (iii) Store Acumulador, PosMemX
```

Esto significa que entre cualquiera de esos pasos, otro proceso podría intervenir y modificar las variables, generando resultados inesperados.

---

### Ejemplos de no determinismo por grano fino

**Ejemplo 1 — 3 procesadores, lecturas y escrituras atómicas:**
```text
x = 0; y = 4; z = 2;
co
  x = y + z          (1)   →  (1.1) Load PosMemY, Acumulador
                               (1.2) Add PosMemZ, Acumulador
                               (1.3) Store Acumulador, PosMemX
  // y = 3           (2)   →  (2) Store 3, PosMemY
  // z = 4           (3)   →  (3) Store 4, PosMemZ
oc
```
- `y = 3` y `z = 4` en todos los casos.
- `x` puede ser: 6, 5, 8, 7, según el orden de ejecución de los pasos.

**Ejemplo 2 — 2 procesadores, lecturas y escrituras atómicas:**
```text
x = 2; y = 2;
co
  z = x + y          (1)   →  (1.1) Load PosMemX, Acumulador
                               (1.2) Add PosMemY, Acumulador
                               (1.3) Store Acumulador, PosMemZ
  // x = 3; y = 4;   (2)   →  (2.1) Store 3, PosMemX
                               (2.2) Store 4, PosMemY
oc
```
- `x = 3` e `y = 4` en todos los casos.
- `z` puede ser: 4, 5, 6 o 7.

> Nunca podríamos pausar el programa y ver un estado donde `x + y = 6`, aunque `z = x + y` sí puede tomar ese valor dependiendo del momento en que se ejecute.

---

### Características del grano fino

- Los valores de los tipos básicos se almacenan en elementos de memoria leídos y escritos como acciones atómicas.
- Los valores se cargan en registros, se opera sobre ellos, y luego se almacenan en memoria.
- Cada proceso tiene su propio conjunto de registros (context switching).
- Todo resultado intermedio de una expresión compleja se almacena en registros o memoria privada del proceso.

Los tiempos absolutos no importan: no se puede confiar en el tiempo de ejecución de las sentencias ni en la velocidad del procesador para sincronizar procesos, ya que los procesadores cambian y el tiempo de cada etapa varía.

---

### Referencia crítica y Propiedad "A lo sumo una vez" (ASV)

Una **referencia crítica** es una referencia a una variable compartida que puede ser modificada por otro proceso.

La propiedad **"A lo sumo una vez" (ASV)** permite que una sentencia se ejecute *como si* fuera atómica, siempre que se cumplan estas condiciones:

Una sentencia `x = e` satisface ASV si:
1. `e` contiene **a lo sumo una referencia crítica** y `x` no es referenciada por otro proceso, **o**
2. `e` **no contiene referencias críticas**, en cuyo caso `x` puede ser leída por otro proceso.

Una expresión `e` que no está en una sentencia de asignación satisface ASV si no contiene más de una referencia crítica.

> En resumen: **puede haber a lo sumo una variable compartida, y puede ser referenciada a lo sumo una vez.**

Si una sentencia cumple ASV, su ejecución *parece* atómica porque la variable compartida será leída o escrita solo una vez.

**Ejemplos:**
```text
int x=0, y=0;
co x=x+1 // y=y+1 oc
→ No hay referencias críticas en ningún proceso.
  En todas las historias: x = 1 e y = 1 ✔

int x=0, y=0;
co x=y+1 // y=y+1 oc
→ El 1er proceso tiene 1 ref. crítica (y). El 2do ninguna.
  Siempre y = 1, y x = 1 o 2 ✔

int x=0, y=0;
co x=y+1 // y=x+1 oc
→ Ninguna asignación satisface ASV.
  Posibles resultados: x=1 e y=2  /  x=2 e y=1
  Nunca debería ocurrir x=1 e y=1 → ERROR ✘
```

> ¿Cuándo `x` valdrá 2 en el segundo ejemplo? Solo si se da esta secuencia:
> 1. Proceso 1: Load X
> 2. Proceso 2: hace N-1 iteraciones del loop
> 3. Proceso 1: incrementa su copia
> 4. Proceso 1: Store X
> 5. Proceso 2: Load X
> 6. Proceso 1: hace el resto de las iteraciones del loop
> 7. Proceso 2: incrementa su copia
> 8. Proceso 2: Store X
>
> **→ No podemos confiar en la intuición para analizar un programa concurrente.**

---

## Especificación de la sincronización

Cuando una expresión o asignación **no satisface ASV**, es necesario ejecutarla atómicamente para evitar resultados incorrectos. En general, se necesita ejecutar secuencias enteras de sentencias como una única acción atómica: esto es la **sincronización por exclusión mutua**.

Para expresar esto, se usan **acciones atómicas de grano grueso** (*coarse grained*): secuencias de acciones de grano fino que, vistas desde afuera, aparecen como indivisibles.

### Sintaxis `<await>`
```text
⟨e⟩               → evalúa la expresión e atómicamente.

⟨await (B) S;⟩    → espera a que B sea verdadera y luego ejecuta S atómicamente.
```

- `B` es una expresión booleana que especifica una **condición de demora**: el proceso espera hasta que se cumpla.
- `S` es una secuencia de sentencias que se garantiza que termina.
- Se garantiza que `B` es `true` cuando comienza la ejecución de `S`.
- **Ningún estado interno de `S` es visible para los otros procesos.**

### Tipos de `await`

| Tipo | Ejemplo | Descripción |
|---|---|---|
| **General** | `⟨await (s>0) s=s-1;⟩` | Espera condición y ejecuta sentencias atómicamente. |
| **Exclusión mutua** | `⟨x = x + 1; y = y + 1⟩` | Sin condición de demora, solo garantiza atomicidad. |
| **Condición** | `⟨await (count > 0)⟩` | Solo espera que se cumpla una condición, sin sentencias. |

Cuando `B` satisface ASV, la sincronización por condición puede implementarse como **busy waiting** o **spin loop**:
```text
do (not B) → skip od     ←→     while (not B);
```

> El `await` es una sentencia de alto poder expresivo, pero su implementación general (con exclusión mutua y condición simultáneas) tiene un costo alto.

### Ejemplo — Productor/Consumidor con buffer de tamaño N

Un caso clásico de sincronización por condición: un proceso produce elementos y los coloca en un buffer, y otro los consume. Deben coordinarse para que el productor no agregue si el buffer está lleno, y el consumidor no saque si está vacío.
```text
cant: int = 0;
Buffer: cola;

process Productor {
    while (true)
        Generar Elemento
        ⟨await (cant < N); push(buffer, elemento); cant++⟩
}

process Consumidor {
    while (true)
        ⟨await (cant > 0); pop(buffer, elemento); cant--⟩
        Consumir Elemento
}
```

- El productor espera a que haya lugar (`cant < N`) antes de agregar al buffer.
- El consumidor espera a que haya elementos (`cant > 0`) antes de sacar del buffer.
- Ambas operaciones se ejecutan atómicamente: nadie puede interferir entre la verificación de la condición y la modificación del buffer.
