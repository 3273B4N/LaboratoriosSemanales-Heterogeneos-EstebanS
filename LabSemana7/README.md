# Laboratorio Semana 7

## Entorno de desarrollo

| Elemento | Valor |
|---|---|
| GPU | NVIDIA Jetson Nano 2GB Developer Kit |
| Driver | Jetpack 4.6.1 [L4T 32.7.1] |
| CUDA Toolkit  | nvcc --version|
| Sistema operativo | Ubuntu 18.04.6 LTS|
| Kernel | Linux 4.9.253-tegra|

CUDA build:

```bash
el5859@jetson-181:~$ nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2021 NVIDIA Corporation
Built on Sun_Feb_28_22:34:44_PST_2021
Cuda compilation tools, release 10.2, V10.2.300
Build cuda_10.2_r440.TC440_70.29663091_0
```



---

## Ejercicio A: Suma de vectores

### Implementación

Se agregó la linea c[i] = a[i] + b[i]; al kernel, 

En el codigo la condicion i < n es necesaria ya que se asigna una suma a cada hilo disponible, esto mientras el numero del hilo (que debe ser el numero de hilos disponibles) sea menor al numero del vector que se va asumar

### Salida de ejecución

``` bash
el5859@jetson-181:~/esteb/LaboratoriosSemanales-Heterogeneos-EstebanS/LabSemana7/vector-add$ ./vector_add

vector-add n=1048576: OK
```

Con otro tamaño (`make run N=2097152`):

```bash
el5859@jetson-181:~/esteb/LaboratoriosSemanales-Heterogeneos-EstebanS/LabSemana7/vector-add$ make run N=2097152

nvcc -O2 -std=c++14 -lineinfo -o vector_add main.cu
./vector_add 2097152
vector-add n=2097152: OK
```

### Preguntas

**1. ¿Cuántos bloques se lanzan cuando N=1048576 y cada bloque tiene 256 hilos?**

Se lanzan 4096 bloques


**2. ¿Qué ocurre si N no es múltiplo del tamaño del bloque?**

Se debe lanzar un bloque extra para poner en threads los vectores sobrantes, lo cual causa subutilizacion debido a que no todos los threads van a estar ocupados, por ejemplo corriendo este programa pero con 
 N=1048577  lanza 4097 bloques, ese bloque extra corresponde al vector extra que no se pudo asignar en el thread anterior por lo que se lanza un bloque entero para ese vector.
 
**3. ¿Qué transferencias de memoria ocurren entre CPU y GPU?**

Se transfiere  h_a y h_b de la CPU a la GPU (en d_b, y d_a), de la GPU a la CPU se transfiere d_c a h_c 

---

## Ejercicio B: Producto punto


### Implementación

Se completo el kernel con el calculo del producto parcial usando: value += a[i] * b[i]; 

Posteriormente se acumulan en la cache con cache[tid] += cache[tid + stride];



### Salida de ejecución

`make run N=1048576`:

```bash
el5859@jetson-181:~/esteb/LaboratoriosSemanales-Heterogeneos-EstebanS/LabSemana7/dot-product$ make run N=1048576
nvcc -O2 -std=c++14 -lineinfo -o dot_product main.cu
./dot_product 1048576
```

`make run N=4194304`:

```bash
el5859@jetson-181:~/esteb/LaboratoriosSemanales-Heterogeneos-EstebanS/LabSemana7/dot-product$ make run N=4194304
./dot_product 4194304
dot-product n=4194304: gpu=-0.500000 cpu=-0.500000 error=0.000000 OK
```

### Preguntas

**1. ¿Por qué este ejercicio no puede resolverse solamente escribiendo un valor independiente por hilo?**

En este programa muchos hilos contribuyen a una sola respuesta, por lo que se deben combinar sus valores, que es lo que hace la reduccion lo cual requiere que los hilos se compartan datos y se sincronicen, pero como los hilos de diferentes bloques  no pueden hacer esto, se calcula un producto parcial y al final se hace una sola suma en la CPU (donde todos los hilos aportan su valor en la memoria global).
  
**2. ¿Cuántos valores parciales se copian de GPU a CPU?**

Se lanzan N/hilos_por_bloque  ya que cada bloque procesa un producto parcial.

**3. ¿Qué pasaría si se elimina alguna sincronización dentro de la reducción?**
Es posible que un bloque no haya terminado de calcular su resultado y que por ende se sume un valor antiguo o corrupto.

**4. Explique el papel de `__syncthreads()`.**

El papel de __syncthreads(); es la de sincronizar los hilos, esto en necesario ya que puede que hayan hilos que terminen primero que otros debido a la diferencia en la carga de trabajo por lo que es necesario sincronizarlos para obtener todos los resultados al mismo tiempo.

---

## Ejercicio C: Softmax

### Implementación

Se completa el kernel con local_max = fmaxf(local_max, input[rows * cols + col]); para calcular el máximo de cada fila. Además de que se completa con cache[tid] = fmaxf(cache[tid], cache[tid+stride]);, para reducir los máximos. También se calcula la exponencial y se acumula con: output[idx]=expf(input[idx] - row_max); y local_sum += output[idx];, se reducen las sumas parciales con cache[tid] += cache[tid +stride];. Por último se normalizó el vector de salida con output[idx]=output[idx]/row_sum;.

### Salida de ejecución

`make run`:

```bash
el5859@jetson-181:~/esteb/LaboratoriosSemanales-Heterogeneos-EstebanS/LabSemana7/softmax$ ./softmax 
softmax rows=128 cols=1024: OK

```

`make run ROWS=256 COLS=2048`:

```bash
el5859@jetson-181:~/esteb/LaboratoriosSemanales-Heterogeneos-EstebanS/LabSemana7/softmax$ make run ROWS=256 COLS=2048
./softmax 256 2048
softmax rows=256 cols=2048: OK
```

### Preguntas

**1. ¿Por qué se calcula primero el máximo de cada fila?**

Debido a que aporta estabilidad numerica (que tan bien un algoritmo resiste los errores de redondeo y los limites de representacion numérica en una computadora) al resultado.

**2. ¿Qué partes del algoritmo requieren cooperación entre hilos del mismo bloque?**

La parte de calcular el máximo requiere cooperacion entre hilos del mismo bloque ya que ningún hilo por si solo puede saber si el valor que esta procesando es el maximo, sino que tiene que saber el valor de los demás. Además de la parte de completar la reducción para sumar las exponenciales debido a que se debe sumar los resultados de cada hilo para tener el valor final. 

**3. ¿Qué limitación tiene usar un solo bloque por fila cuando cols crece mucho?**

Que los bloques tienen un numero máximo de hilos, por lo que no se pueden poner un número arbitrario de cols, sino que esta limitado al número máximo de hilos por bloque.

---



<!-- observaciones, dificultades, aprendizajes -->
