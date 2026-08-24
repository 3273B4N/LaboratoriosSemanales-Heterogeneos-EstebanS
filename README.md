# LaboratoriosSemanales-Heterogeneos-EstebanS

# LabSemana3
## Parte D: Contraste de tiempos de ejecución

Multiplicación de matrices 1024×1024, comparando versión escalar vs. vectorizada (AVX2).

| Versión | Tiempo (s) | Rendimiento (GFLOP/s) |
|---------|------------|------------------------|
| Escalar | 0.816657   | 2.629604               |
| AVX2    | 0.213835   | 10.042729              |

**Speedup:** ≈ 3.82×

El speedup obtenido está por debajo del máximo teórico de 8× (AVX2 procesa 8 floats por instrucción), principalmente por el trabajo extra que introduce la reducción horizontal en el producto punto y por limitaciones de acceso a memoria/caché.
