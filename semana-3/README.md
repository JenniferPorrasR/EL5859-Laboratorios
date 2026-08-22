# Laboratorio semana 3 - Instrucciones SIMD (AVX2)

Jennifer Porras Rojas 

## Ejercicio A

```c
static inline __m256 simd_mul_ps(__m256 a, __m256 b)
{
   // inciso A
    return _mm256_mul_ps(a, b);
}
```

## Ejercicio B

```c
static inline float simd_reduce_add_ps(__m256 value)
{
    //inciso B
    __m128 low = _mm256_extractf128_ps(value, 0);
    __m128 high = _mm256_extractf128_ps(value, 1);
    __m128 sum = _mm_add_ps(low, high);

    sum = _mm_hadd_ps(sum, sum);
    sum = _mm_hadd_ps(sum, sum);

    return _mm_cvtss_f32(sum);
}
```

## Ejercicio C

```c
float dot_product_avx2(const float a[VECTOR_SIZE], const float b[VECTOR_SIZE])
{
    float result = 0.0f;
    // inciso C
    for (int i = 0; i < VECTOR_SIZE; i += AVX_FLOATS) {
        __m256 vec_a = simd_loadu_ps(&a[i]);
        __m256 vec_b = simd_loadu_ps(&b[i]);
        
        __m256 vec_mul = simd_mul_ps(vec_a, vec_b);
        result += simd_reduce_add_ps(vec_mul);
    }

    return result;
}
```


## Ejercicio D

Al ejecutar la versión vectorizada y la no vectorizada se obtuvieron los siguientes resultados:

| Versión  | Tiempo (s) | Rendimiento (GFLOP/s) | Checksum            |
|----------|-----------|------------------------|----------------------|
| Escalar  | 7.513694  | 2.286475                | 86972906452.000000  |
| AVX2     | 2.400345  | 7.157251                | 86972906452.000000  |

Ambas versiones producen el mismo checksum y los mismo valores en 'C[0][0]' y en 'C[1023][1023]', con un speedup de aproximadamente 3.13x, lo que indica que la versión vectorizada no introduce errores, si no que solo mejora el tiempo.
