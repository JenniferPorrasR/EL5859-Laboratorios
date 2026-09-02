# Práctica de clase 3

Jennifer Porras Rojas 2020112477

## Características del procesador 

| Ítem | Detalle |
|---|---|
| Procesador | 11th Gen Intel(R) Core (TM) i7-1165G7 |
| Núcleos físicos / hilos (SMT) | 4 / 2 |
| CPU(s) | 8|



## Ejercicio A

### Resultados cpu-affinity

**Gráfico de tiempo vs número de hilos**

[![tiempo-affinity.png](https://i.postimg.cc/9XDNvJxh/tiempo-affinity.png)](https://postimg.cc/t7H56tQS)

**Gráfico de Speedud vs número de hilos**

[![speedup-affinity.png](https://i.postimg.cc/dVw6tB92/speedup-affinity.png)](https://postimg.cc/fJqm8Ypy)

**Gráfico de eficiencia vs número de hilos**

[![eficiencia-affinity.png](https://i.postimg.cc/x1fGSd0m/eficiencia-affinity.png)](https://postimg.cc/F76kyNkF)

**Escalabilidad**

El speedup alcanzado nunca es mayor que 1.0: añadir hilos, en casi todos los casos, no acelera el programa, sino que lo hace más lento. La duración en tiempo real aumenta de manera casi constante desde 2.48 s (1 hilo) hasta un pico de 3.29 s (6-7 hilos), con una ligera mejora al llegar a los 8 hilos. Esto señala una escalabilidad negativa: el trabajo útil que cada hilo lleva a cabo es menor que el costo de crear, sincronizar y unir los hilos. El programa no escala porque la carga de trabajo es insignificante en comparación con el overhead que implica gestionar los hilos.

**Proporción de código paralelo**

Utilizando la ley de Amdahl S(N) = 1 / ( (1 - P) + P/N ) y despejando P a partir del dato medido en en N = 2 se obtiene:

0.96 = 1 / ((1-P) + P/2)

(1-P) + P/2 = 1.0417

1 - P/2 = 1.0417

P ≈ -0.083

Se obtiene una fracción paralela negativa, lo que en el modelo de Amdahl no tiene sentido. Esto señala que el resultado no puede ser explicado solamente por la sección en serie del algoritmo. El modelo de Amdahl no tiene en cuenta el costo extra de la creación y sincronización de hilos, que es lo que más afecta el rendimiento en este caso. Por ende, para un problema de tal magnitud, paralelizar el algoritmo no es prácticamente ventajoso porque la mejora lograda está cerca del 0%.

**Eficiencia**

La eficiencia reduce rápidamente: con 1 hilo es del 100%, con 2 hilos del 48% y con 8 hilos se aproxima al 10%. Esto sucede porque el costo de generar y organizar los hilos puede ser más alto que la labor que cada uno desempeña. El tamaño de los datos es pequeño, por lo que emplear múltiples hilos no produce una mejora significativa. Cuando se incrementa el número de hilos que están trabajando simultáneamente, puede existir competencia por recursos del procesador, tales como la caché y la memoria.

### Resultados cpu-naive

**Gráfico de tiempo vs número de hilos**

[![tiempo-naive.png](https://i.postimg.cc/Hn5LKbcx/tiempo-naive.png)](https://postimg.cc/308704Z5)

**Gráfico de Speedud vs número de hilos**

[![speedup-naive.png](https://i.postimg.cc/wj9Bfy67/speedup-naive.png)](https://postimg.cc/hf3ghGQB)

**Gráfico de eficiencia vs número de hilos**

[![eficiencia-naive.png](https://i.postimg.cc/Jh24zpj1/eficiencia-naive.png)](https://postimg.cc/WD0P969K)

**Escalabilidad**

El speedup nunca sobrepasa 1.0: añadir hilos no acelera el programa, sino que lo hace cada vez más lento. El tiempo real aumenta de manera monótona desde 2.48 s (1 hilo) hasta 3.82 s (8 hilos), sin que haya un solo punto en el que se observe una mejora. Esto señala una escalabilidad negativa más pronunciada que en la versión con afinidad: si los hilos no están asignados a núcleos concretos, el sistema operativo tiene la capacidad de trasladarlos entre núcleos mientras se ejecutan, lo cual provoca un incremento del overhead de migración, pérdida de localidad de caché y una mayor contención. El costo de crear, planificar y sincronizar los hilos siempre es más elevado que el trabajo útil que cada uno de ellos hace.

**Proporción de código paralelo**

Utilizando la ley de Amdahl S(N) = 1 / ( (1 - P) + P/N ) y despejando P a partir del dato medido en en N = 2 se obtiene:

0.88 = 1 / ((1-P) + P/2)

(1-P) + P/2 = 1.1364

P ≈ -0.273

Una vez más, se obtiene una fracción paralela negativa, lo que demuestra que el modelo de Amdahl no es capaz de explicar la conducta observada: el costo dominante no es la parte en serie del algoritmo, sino el overhead de gestión de hilos, que en este caso se ve agravado por la falta de afinidad con la CPU. En la práctica, el porcentaje de código que vale la pena paralelizar para este tamaño de problema es prácticamente 0% y es incluso más bajo que en la versión con `cpu-affinity`.

**Eficiencia**

La eficiencia baja con rapidez: baja del 100% con un hilo al 44% con dos hilos y se reduce a cerca del 8% cuando hay ocho hilos. Esta disminución es más significativa que la lograda con `cpu-affinity`, lo que sugiere que, al asignar los hilos a núcleos específicos, el rendimiento se mejora ligeramente. No obstante, la tendencia continúa siendo desfavorable. Esto se debe principalmente a que el costo de crear y eliminar hilos es más alto que la labor que cada uno realiza. Además, la magnitud de los datos es reducida para beneficiarse del paralelismo, y dado que no se gestiona en qué núcleo se ejecuta cada hilo, la competencia por recursos como el bus, la memoria y la caché crece.

## Ejercicio B

### Resultados matmul_tiled_openmp

**Gráfico de tiempo vs número de hilos**

[![tiempo-matmul-tiled.png](https://i.postimg.cc/3J56H207/tiempo-matmul-tiled.png)](https://postimg.cc/9zb1dRBN)

**Gráfico de Speedud vs número de hilos**

[![speedup-matmul-tiled.png](https://i.postimg.cc/6pBm8whD/speedup-matmul-tiled.png)](https://postimg.cc/9rNpk6Kp)

**Gráfico de eficiencia vs número de hilos**

[![eficiencia-matmul-tiled.png](https://i.postimg.cc/g0JtKDQB/eficiencia-matmul-tiled.png)](https://postimg.cc/G4WjLkck)

**Escalabilidad**

En contraste con los ejercicios previos, en este caso se nota una escalabilidad positiva: el speedup aumenta de 1.00 a 4.02 y el tiempo de ejecución disminuye de 0.735 s (1 hilo) a 0.183 s (8 hilos). Esto tiene sentido, ya que `matmul-tiled` presenta una carga de trabajo por hilo considerablemente más alta (multiplicación de matrices con *tiling*), lo cual hace que el trabajo útil compense en gran medida los gastos generales de crear y sincronizar hilos, algo que no sucedía con `cpu-affinity`/`cpu-naive`.

No obstante, la escalabilidad **no es lineal**: se observa una caída en 5 hilos (la velocidad baja de 3.33 a 2.53), después se recupera la tendencia ascendente hasta llegar a los 8 hilos. Esto indica un problema de **balanceo de carga**, ya que, si la matriz no se distribuye equitativamente entre cinco hilos, algunos de ellos obtienen más tiles que otros, lo cual crea un desequilibrio que afecta negativamente el tiempo total (el hilo más lento es quien determina dicho tiempo). La división del trabajo es más equitativa cuando se utilizan 4, 6, 7 u 8 hilos, lo que justifica la recuperación de la velocidad.

**Proporción de código paralelo**

Utilizando la ley de Amdahl S(N) = 1 / ( (1 - P) + P/N ) y despejando P a partir del dato medido en en N = 2 se obtiene:

1.88 = 1 / ((1-P) + P/2)

(1-P) + P/2 = 0.5319

P ≈ 0,9362

Si se despeja P usando el dato medida en N = 8 se obtiene:

4.02 = 1 / ((1-P) + P/2)

(1-P) + P/2 = 0.2488

P ≈ 0.859

Se observa que, en contraste con los ejercicios A con `cpu-naive`/`cpu-affinity`, **hay una porción del código que puede ser paralelizada de manera significativa (entre 85% y 94%)**. Ambas estimaciones están alineadas entre sí. El modelo de Amdahl es ideal y no tiene en cuenta efectos reales, como el desbalanceo de carga que se observa en cinco hilos o el overhead de sincronización que aumenta a medida que el número de hilos crece; por eso, las dos estimaciones son ligeramente diferentes.

**Eficiencia**

La eficiencia se reduce de manera progresiva: desde el 100 % (1 hilo) hasta el 83 % (4 hilos), lo que es un buen indicador de paralelización efectiva. No obstante, desciende abruptamente a cerca del 51% en cinco hilos, lo que coincide con la anomalía de speedup, y posteriormente se estabiliza entre el 50 y el 55% en seis y ocho hilos. Esta estabilización, que es relativamente alta (en comparación con las ejecuciones de `cpu-naive`, que bajaban a aproximadamente el 10%), demuestra que el programa tiene una carga computacional real y utilizable en paralelo. Además, señala que la disminución de la eficiencia se debe principalmente a que los hilos adicionales compiten por los mismos recursos de ejecución y caché (memoria compartida) al exceder el número de núcleos físicos del procesador, en vez de contribuir con cómputo genuinamente adicional.

### Resultados softmax_openmp

**Gráfico de tiempo vs número de hilos**

[![tiempo-softmax.png](https://i.postimg.cc/NGSyGjQv/tiempo-softmax.png)](https://postimg.cc/5j59n1Hs)

**Gráfico de Speedud vs número de hilos**

[![speedup-softmax.png](https://i.postimg.cc/65Z88xQR/speedup-softmax.png)](https://postimg.cc/BXqSzyHQ)

**Gráfico de eficiencia vs número de hilos**

[![eficiencya-softmax.png](https://i.postimg.cc/pLYyBQzr/eficiencya-softmax.png)](https://postimg.cc/18X9mqW1)

**Escalabilidad**

En contraste con `matmul-tiled`, en este caso el programa **obtiene algo de paralelismo, pero es muy limitado y no se mantiene a lo largo del tiempo**. El tiempo disminuye de 0.62 s (1 hilo) a un mínimo de 0.42 s (4 hilos), lo que resulta en un speedup máximo de solo 1.47x. A partir de 5 hilos, el tiempo vuelve a aumentar de manera continua hasta alcanzar los 0.65 segundos con 8 hilos. Esto hace que el speedup se reduzca por debajo de 1.0 (0.96), lo cual significa que **el programa termina siendo más lento con 8 hilos que con uno solo**.

**Proporción de código paralelo**

Utilizando la ley de Amdahl S(N) = 1 / ( (1 - P) + P/N ) y despejando P a partir del dato medido en en N = 2 se obtiene:

1.37 = 1 / ((1-P) + P/2)

(1-P) + P/2 = 0.270

P ≈ 0,54

Si se despeja P usando el dato medida en N = 4 se obtiene:

1.47 = 1 / ((1-P) + P/2)

(1-P) + P/4 = 0.6803

P ≈ 0.427

Las dos estimaciones son coherentes y reflejan un porcentaje paralelo **moderado (entre el 43% y el 54%)**, el cual es considerablemente más bajo que en `matmul-tiled` (85–94%), pero es evidentemente más alto que en `cpu-naive`/`cpu-affinity` (~0%). Esto es consistente con la naturaleza del algoritmo: el softmax cuenta con una fase que puede ejecutarse de manera paralela (el cálculo de exponenciales) y otra que debe ser secuencial (la suma total para normalizar), lo cual reduce la porción paralelizable. Esta última se vuelve más costosa en términos de sincronización a medida que el número de hilos crece.

**Eficiencia**

La eficiencia desciende de manera rápida y constante en todo el rango, desde 100 % (1 hilo) a 69 % (2 hilos), 48 % (3 hilos), y continúa disminuyendo hasta llegar a solo el 12 % con 8 hilos. Incluso en el mejor caso de speedup (4 hilos), la eficiencia es solo del 37%, lo que demuestra que, incluso en su mejor momento, el paralelismo todavía está lejos de ser óptimo. La poca eficiencia se debe a que el tamaño de la carga de trabajo por hilo es pequeño, a la contención de memoria compartida cuando se excede el número de núcleos físicos disponibles y al overhead de sincronización en la fase de reducción (suma para normalizar el softmax). Por estas razones, añadir más hilos después del cuarto resulta perjudicial para este problema.


# Práctica de clase 3

Jennifer Porras Rojas 2020112477

## Características del procesador 

| Ítem | Detalle |
|---|---|
| Procesador | 11th Gen Intel(R) Core (TM) i7-1165G7 |
| Núcleos físicos / hilos (SMT) | 4 / 2 |
| CPU(s) | 8|

## Ejercicio A

### Comando ejecutado 

```bash
./libraries/build/bin/bench-static 1000000 1000 1.0 2.0
```

### Resultado 

| Operación | Tiempo total (µs) | Tiempo por iteración (µs) |
|---|---|---|
| fill A | 460,327.077 | 460.327 |
| fill B | 467,993.806 | 467.994 |
| add    | 925,004.951 | 925.005 |
| **Total** | **1,853,326.521** | — |


### Comando ejecutado 

```bash
ls -lh libraries/build/lib/libvectorops.a
```

| Archivo | Tamaño |
|---|---|
| libvectorops.a | 1.8K |


## Ejercicio B

### Comando ejecutado 

```bash
./libraries/build/bin/bench-dynamic 1000000 1000 1.0 2.0
```

### Resultado 

| Operación | Tiempo total (µs) | Tiempo por iteración (µs) |
|---|---|---|
| fill A | 1,500,890.346 | 1,500.890 |
| fill B | 1,578,454.028 | 1,578.454 |
| add    | 1,531,545.902 | 1,531.546 |
| **Total** | **4,610,890.998** | — |

### Comando ejecutado 

```bash
ls -lh libraries/build/lib/libvectorops.so
```

| Archivo | Tamaño |
|---|---|
| libvectorops.so | 16K |




