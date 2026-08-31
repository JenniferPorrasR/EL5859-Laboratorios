# Práctica de clase 3

Jennifer Porras Rojas 2020112477

## Características del procesador 

| Ítem | Detalle |
|---|---|
| Procesador | 11th Gen Intel(R) Core (TM) i7-1165G7 |
| Núcleos físicos / hilos (SMT) | 4 / 2 |
| CPU(s) | 8|



## Ejercicio A

### Resultados

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

