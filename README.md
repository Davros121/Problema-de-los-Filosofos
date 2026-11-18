# Problema-de-los-Filosofos
Trabajo calificado de Sistemas Operativos II, Ing. en Sistemas. En la presente se desarrolla el problema de la cena de los filosofos resuelto en Lenguaje C, empleando el uso de Hilos POSIX (Pthreads) y Semaforos (semaphore.h)

Integrantes:
Jesus Ernesto Ramirez Ruiz - 28.661.648
Anthony Jose Alvarado - 28.

La Cena de los Filósofos es una metáfora sobre 5 procesos (filósofos) que compiten por 5 recursos compartidos (tenedores) en una mesa redonda. Cada uno necesita dos tenedores simultáneamente para comer, pero si todos toman el de su izquierda al mismo tiempo, ninguno podrá tomar el de su derecha y el sistema se congela por completo (interbloqueo o deadlock).

Por que es imporatnte comprenderlo?:
Enseña a coordinar múltiples tareas que se ejecutan en paralelo sobre recursos limitados (memoria, archivos, bases de datos). Dominar su solución (mediante semáforos y asimetría) garantiza tres cosas en el desarrollo de software real:

  * Evitar bloqueos: Impide que los programas se queden congelados esperando recursos retenidos por otros procesos.

  * Proteger los datos: Asegura que dos procesos no modifiquen la misma información al mismo tiempo, evitando corrupción.

  * Garantizar equidad: Permite que todos los procesos avancen sin quedar relegados indefinidamente (inanición).
    

    ## **Guía paso a paso: Desarrollo en C sobre Fedora Linux**

**- Paso 1:** Crear el entorno en terminal
Abre la terminal en Fedora y crea un directorio limpio para trabajar:

```
Bash
mkdir proyecto_filosofos
cd proyecto_filosofos
nano filosofos.c
```

**- Paso 2:** Escribir el código en nano
  Copia las siguientes tres partes en orden dentro del archivo. La estrategia aplicada aquí es romper la simetría: los primeros 4 filósofos toman el tenedor izquierdo primero, pero el quinto toma primero el derecho, eliminando la posibilidad de interbloqueo.

*Parte A:* Librerías y recursos compartidos

```
C
#include <stdio.h>      // Entrada y salida estándar (printf)
#include <stdlib.h>     // Números aleatorios (rand)
#include <pthread.h>    // Manejo de hilos (filósofos actuando en paralelo)
#include <semaphore.h>  // Semáforos (control de acceso a los tenedores)
#include <unistd.h>     // Pausas de tiempo (sleep)

#define TOTAL 5
sem_t tenedores[TOTAL]; // Cada semáforo representa un tenedor (1 = libre, 0 = ocupado)
```

*Parte B:* Ciclo de vida de cada filósofo
```
C
void* accion_filosofo(void* numero) {
    int id = *(int*)numero;           // Número del filósofo (0 al 4)
    int izq = id;                     // Tenedor a su izquierda
    int der = (id + 1) % TOTAL;       // Tenedor a su derecha

    while (1) {
        // 1. Pensar
        printf("Filosofo %d: Pensando...\n", id);
        sleep(rand() % 3 + 1);

        // 2. Intentar tomar ambos tenedores
        if (id == TOTAL - 1) {
            // El último filósofo invierte el orden para evitar el deadlock
            sem_wait(&tenedores[der]);
            sem_wait(&tenedores[izq]);
        } else {
            sem_wait(&tenedores[izq]);
            sem_wait(&tenedores[der]);
        }


        // 3. Comer (Sección crítica: tiene ambos recursos asegurados)
        printf("Filosofo %d: COMIENDO con tenedores %d y %d\n", id, izq, der);
        sleep(rand() % 3 + 1);

        // 4. Liberar ambos tenedores para los vecinos
        sem_post(&tenedores[izq]);
        sem_post(&tenedores[der]);
    }
}
```

Parte C: Inicialización del programa (main)
```
C
int main() {
    pthread_t filosofos[TOTAL];
    int ids[TOTAL];

    // Inicializar los 5 tenedores como disponibles (valor 1)
    for (int i = 0; i < TOTAL; i++) {
        sem_init(&tenedores[i], 0, 1);
        ids[i] = i;
    }

    // Lanzar los 5 hilos en paralelo
    for (int i = 0; i < TOTAL; i++) {
        pthread_create(&filosofos[i], NULL, accion_filosofo, &ids[i]);
    }

    // Mantener el hilo principal activo mientras los filósofos se ejecutan
    for (int i = 0; i < TOTAL; i++) {
        pthread_join(filosofos[i], NULL);
    }

    return 0;
}
```
**Paso 3:** Guardar, compilar y ejecutar
Guardar en nano: Presiona Ctrl + O, luego Enter. Sal con Ctrl + X.

Compilar: Enlaza la librería POSIX Threads con -lpthread:

``` bash
gcc filosofos.c -o filosofos -lpthread
```

Ejecutar:
``` bash
./filosofos
```
Detén la ejecución en cualquier momento presionando Ctrl + C.
