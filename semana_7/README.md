# Práctica semana 7

Jennifer Porras Rojas 2020112477

## Equipo utilizado 

| Componente | Especificación |
|---|---|
| Modelo | NVIDIA Jetson Nano 2GB Developer Kit |
| JetPack / L4T | JetPack 4.6.1 (L4T R32.7.1) |
| Sistema operativo | Ubuntu 18.04.6 LTS (kernel 4.9.253-tegra) |
| CPU | ARM Cortex-A57, 4 núcleos (aarch64) |
| Memoria | 2 GB LPDDR4, compartida entre CPU y GPU |
| GPU | NVIDIA Maxwell, 128 núcleos CUDA (1 SM), compute capability 5.3 |
| CUDA | 10.2 (nvcc V10.2.300) |

## Ejercicio A: suma de vectores 

Se completó el kernel `vector_add_kernel` que calcula `C[i] = A[i] + B[i]`:

```cuda
int i = blockIdx.x * blockDim.x + threadIdx.x;

    if (i < n) {
        // TODO: Calcule c[i] = a[i] + b[i].
	    c[i] = a[i] + b[i];
    }
}
```

### Salidas

```
$ make run
./vector_add 1048576
vector-add n=1048576: OK

$ make run N=1000
./vector_add 1000
vector-add n=1000: OK
```

### Preguntas 

**¿Cuántos bloques se lanzan cuando N=1048576 y cada bloque tiene 256 hilos?**

La cantidad de bloques se calcula al dividr N entre el tamaño del bloque, por lo que:

Número de bloques = 1 048 576 / 256 = 4096 bloques

Se inician, en total, 1 048 576 hilos (4096 × 256), que es igual al número de elementos.

**¿Qué ocurre si N no es múltiplo del tamaño del bloque?**

El programa determina la cantidad de bloques usando la fórmula `(n + threads_per_block - 1) / threads_per_block`, redondeando hacia arriba; por lo tanto, se lanza un bloque extra para abarcar los elementos que faltan. Ese último bloque queda parcialmente ocupado, y algunos hilos obtienen un índice `i ≥ n` que no corresponde a ningún elemento. La condición `if (i < n)` impide que esos hilos se salgan de los arreglos, lo cual podría provocar resultados equivocados o fallos de memoria.

**¿Qué transferencias de memoria ocurren entre CPU y GPU?**

Se reserva memoria en la GPU para los tres vectores utilizando `cudaMalloc` antes de realizar las transferencias. Después, los vectores de entrada A y B se transfieren desde la CPU a la GPU usando `cudaMemcpy(..., cudaMemcpyHostToDevice)`, porque el kernel requiere esos datos en la memoria del dispositivo. El vector C no se envía a la GPU porque su escritura es exclusiva allí; en cambio, con `cudaMemset`, se inicializa directamente en la GPU con valor cero. No se producen transferencias mientras se ejecuta el kernel y al finalizar, se utiliza `cudaMemcpy(..., cudaMemcpyDeviceToHost)` para copiar el vector resultado C desde la GPU a la CPU con el fin de verificarlo. 

## Ejercicio B: producto punto 

En este ejercicio se completó el kernel de producto punto, en donde cada hilo calcula el producto local a[i] * b[i] y luego se realiza la reducción dentro del bloque usando memoria compartida para obtener una suma parcial y la CPU se encarga de sumarlas para obtener el resultado final. 


```cuda
	float value = 0.0f;
    if (i < n) {
        // TODO: Calcule el producto local a[i] * b[i].
	    value = a[i] * b[i];
    }

	cache[tid] = value;
    __syncthreads();

    for (int stride = blockDim.x / 2; stride > 0; stride >>= 1) {
        if (tid < stride) {
            // TODO: Acumule en cache[tid] el valor de cache[tid + stride].
		cache[tid] += cache[tid + stride];
        }
        __syncthreads();
    }

```

### Papel de __syncthreads()

`__syncthreads()` es un método de sincronización para los hilos que pertenecen a un mismo bloque, en donde ningún hilo podrá seguir hasta que todos los hilos del bloque hayan alcanzado ese punto. En este ejercicio, se utiliza en dos ocasiones. Primero, para garantizar que la memoria compartida esté completa antes de comenzar la reducción, cada hilo escribe su producto local en `cache`. En segundo lugar, para asegurar que todas las sumas de cada fase de la reducción se completen antes de que la etapa siguiente lea sus resultados. Sin esto, un hilo podría leer valores de la memoria que otro hilo todavía no ha escrito, generando resultados incorrectos. 


### Salidas 

```
$ make run N=1048576
./dot_product 1048576
dot-product n=1048576: gpu=-21.250000 cpu=-21.250000 error=0.000000 OK

$ make run N=4194304
./dot_product 4194304
dot-product n=4194304: gpu=-0.500000 cpu=-0.500000 error=0.000000 OK
```

### Preguntas 

**¿Por qué este ejercicio no puede resolverse solamente escribiendo un valor independiente por
hilo?**

En la suma de vectores, cada hilo genera su propio resultado (C[i]) de manera independiente. Por otro lado, en el producto punto, el resultado es un solo valor que depende de todos los productos `a[i] * b[i]`. Se produciría una condición de carrera si todos los hilos sumaran directamente a la misma variable, ya que diversos hilos leerían el mismo valor antiguo y se sobrescribirían mutuamente, produciendo un resultado erróneo. Es por esto que se requiere una disminución, en donde cada bloque combina sus productos de manera coordinada en memoria compartida, produce una suma parcial y, posteriormente, la CPU los suma para conseguir el resultado final.

**¿Cuántos valores parciales se copian de GPU a CPU?**

Como el hilo 0 de cada bloque escribe una única suma en `partials[blockIdx.x]`, se copia un valor parcial por bloque. Con 256 hilos por bloque, el número de valores parciales es N / 256:

- Con N = 1 048 576 hay 4096 valores parciales
- Con N = 4 194 304 hay 16 384 valores parciales

**¿Qué pasaría si se elimina alguna sincronización dentro de la reducción?**

Un hilo podría leer `cache[tid + stride]` antes de que el hilo que lo está escribiendo haya terminado, o mientras todavía se actualiza en la fase anterior de la reducción. Lo que generaría una condición de carrera, en donde se sumarían valores viejos o incompletos y el resultado sería erróneo.

## Ejercicio C: softmax

En esre ejercicio cada bloque procesa una fila de la amtriz en cinco pasos:

- Cada hilo calcula el máximo local de sus columnas.
- Se realiza una reducción con fmaxf para obtener el máximo de la fila.
- Cada hilo calcula exponenciales desplazadas, las guarda en output y acumula su suma local.
- Se realiza una segunda reducción en memoria compartida para obtener la suma de las exponenciales.
- Cada hilo normaliza sus elementos dividiendo entre la suma de la fila.


```cuda
    float local_max = -INFINITY;
    for (int col = tid; col < cols; col += blockDim.x) {
        // TODO: Actualice local_max con el maximo de la fila.
	    local_max = fmaxf(local_max, input[row * cols + col]);
    }

    cache[tid] = local_max;
    __syncthreads();

    for (int stride = blockDim.x / 2; stride > 0; stride >>= 1) {
        if (tid < stride) {
            // TODO: Reduzca los maximos usando fmaxf.
	cache[tid] = fmaxf(cache[tid], cache[tid + stride]);
        }
        __syncthreads();
    }

    float row_max = cache[0];

    float local_sum = 0.0f;
    for (int col = tid; col < cols; col += blockDim.x) {
        int idx = row * cols + col;
        // TODO: Calcule expf(input[idx] - row_max), guardelo en output[idx]
        // y acumule el valor en local_sum.
	output[idx] = expf(input[idx] - row_max);
	local_sum += output[idx];
    }

    cache[tid] = local_sum;
    __syncthreads();

    for (int stride = blockDim.x / 2; stride > 0; stride >>= 1) {
        if (tid < stride) {
            // TODO: Reduzca las sumas parciales.
		cache[tid] += cache[tid + stride];
        }
        __syncthreads();
    }

    float row_sum = cache[0];

    for (int col = tid; col < cols; col += blockDim.x) {
        int idx = row * cols + col;
        // TODO: Normalice output[idx] dividiendo entre row_sum.
        output[idx] = output[idx] / row_sum;
    }
}

```


### Salidas 

```
$ make run
./softmax 128 1024
softmax rows=128 cols=1024: OK

$ make run ROWS=256 COLS=2048
./softmax 256 2048
softmax rows=256 cols=2048: OK
```

### Preguntas 

**¿Por qué se calcula primero el máximo de cada fila?**

Por estabilidad numérica, ya que la función exponencial crece con rapidez (en `float`, `expf(x)` se desborda a infinito cuando x excede aproximadamente 88, y el resultado sería `inf/inf = NaN`). Cuando se le resta el máximo, el exponente más alto de la fila se convierte en 0 (e⁰ = 1) y todos los demás son negativos, por lo que, todas las exponenciales quedan entre 0 y 1 y nunca se desbordan. El resultado matemáticamente no se altera, ya que el factor e^(−m) está presente tanto en el numerador como en el denominador y se elimina. Además, dado que el término mayor es igual a 1, la suma nunca puede ser cero.

**¿Qué partes del algoritmo requieren cooperación entre hilos del mismo bloque?**

Las dos reducciones: la suma de las exponenciales y el cálculo del máximo de la fila. Cada hilo únicamente procesa algunas columnas, por lo que para conseguir un valor de la fila completa, los hilos tienen que fusionar sus resultados parciales en una memoria compartida y sincronizarse con `__syncthreads()`. Además, todos los hilos del bloque deben compartir ese resultado, ya que para calcular sus exponenciales, todos requieren el máximo y para normalizar, todos necesitan la suma. 

**¿Qué limitación tiene usar un solo bloque por fila cuando cols crece mucho?**

La cantidad de hilos por bloque es constante (256 en este programa, con un límite máximo de 1024 en CUDA) y todos funcionan en un único multiprocesador (SM). Cada hilo necesita procesar más columnas en secuencia si cols aumenta (4 columnas cuando cols es igual a 1024 y 8 cuando es igual a 2048). De este modo, la labor de cada fila se vuelve más serial y queda restringida a los recursos de un único SM. Asimismo, dado que el número de bloques solo depende de la cantidad de filas, una gran parte de la GPU queda inactiva cuando hay pocas filas y muchas columnas.
