# Práctica semana 7

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

### Salidad 

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

Cuando N no es múltiplo del tamaño del bloque, se genera un bloque adicional para cubrir lo que queda, determinando la cantidad de bloques mediante la fórmula (N + 255) / 256. Como resultado de que ese último bloque queda parcialmente ocupado, ciertos hilos alcanzan un índice i ≥ N que no corresponde con ningún elemento. La condición if (i < n) impide que esos hilos salgan de los arreglos, lo cual podría dar lugar a errores en la memoria o resultados equivocados.

**¿Qué transferencias de memoria ocurren entre CPU y GPU?**

Se ejecutan dos transferencias a la GPU y una de vuelta. Los vectores de entrada A y B son transferidos desde la CPU a la GPU usando cudaMemcpy(..., cudaMemcpyHostToDevice). Al finalizar el kernel, se utiliza cudaMemcpy(..., cudaMemcpyDeviceToHost) para copiar el vector resultado C de la GPU a la CPU. Como en la GPU solo se escribe el vector C, no se transfiere a dicha unidad.

## Ejercicio B: producto punto 

