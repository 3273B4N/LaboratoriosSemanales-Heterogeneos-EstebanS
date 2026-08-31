# Practica 3
## Parte A
```mermaid
xychart-beta
    title "Tiempo real vs número de hilos CPUAffinity"
    x-axis [1, 2, 3, 4, 5, 6]
    y-axis "Tiempo (s)" 0 --> 16
    line [2.49,4.91,7.47,9.99,12.63,15.12]
```

```mermaid
xychart-beta
    title "Tiempo real vs número de hilos CPUNaive"
    x-axis [1, 2, 3, 4, 5, 6]
    y-axis "Tiempo (s)" 0 --> 16
    line [2.46,4.92,7.41,9.88,12.33,14.77]
```

## Parte B

```mermaid
xychart-beta
    title "Tiempo real vs número de hilos MatmulTiled"
    x-axis [1, 2, 3, 4, 5, 6]
    y-axis "Tiempo (s)" 0 --> 0.7
    line [0.500207,0.279048,0.181319,0.156536,0.140146,0.127434]
```

```mermaid
xychart-beta
    title "Tiempo real vs número de hilos SoftMax"
    x-axis [1, 2, 3, 4, 5, 6]
    y-axis "Tiempo (s)" 0 --> 0.7
    line [0.676598,0.59244,0.586893,0.557763,0.675848,0.675848]
```
# Practica 4
## Parte A, B: Tiempos de ejecución

| Versión         | Tiempo de llenado (us) | Tiempo de suma (us) |
|-----------------|------------------------|---------------------|
| bench-static    |         1574785.272    |   3820404.482       |
| bench-dynamic   |         2186325.432    |   5798714.192       |

Se observa que el bench statico tuvo un mejor performance las 2 categorias (menor tiempo de ejecución), esto es debido a que las bibliotecas estáticas son código que se "pega" directamente en el ejecutable, mientras que las bibliotecas dinámicas son accedidas mediantes saltos a otras direcciones de memoria donde se encuentra el ejecutable de la biblioteca, lo que causa el aumento en el tiempo de ejecución.

