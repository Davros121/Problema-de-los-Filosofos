# Problema-de-los-Filosofos
Trabajo calificado de Sistemas Operativos II, Ing. en Sistemas. En la presente se desarrolla el problema de la cena de los filosofos resuelto en Lenguaje C, empleando el uso de Hilos POSIX (Pthreads) y Semaforos (semaphore.h)

Integrantes:
Jesus Ernesto Ramirez Ruiz - 28.661.648

Anthony Jose Alvarado - 28.759.193

La Cena de los Filósofos es una metáfora sobre 5 procesos (filósofos) que compiten por 5 recursos compartidos (tenedores) en una mesa redonda. Cada uno necesita dos tenedores simultáneamente para comer, pero si todos toman el de su izquierda al mismo tiempo, ninguno podrá tomar el de su derecha y el sistema se congela por completo (interbloqueo o deadlock).

**Por que es importante comprenderlo?:**

Enseña a coordinar múltiples tareas que se ejecutan en paralelo sobre recursos limitados (memoria, archivos, bases de datos). Dominar su solución (mediante semáforos y asimetría) garantiza tres cosas en el desarrollo de software real:


  * Evitar bloqueos: Impide que los programas se queden congelados esperando recursos retenidos por otros procesos.

  * Proteger los datos: Asegura que dos procesos no modifiquen la misma información al mismo tiempo, evitando corrupción.

  * Garantizar equidad: Permite que todos los procesos avancen sin quedar relegados indefinidamente (inanición).
    

    ## **Guía paso a paso: Desarrollo en C sobre Fedora Linux**

**- Paso 1:** Crear el entorno en terminal

Abre la terminal en Fedora y crea un directorio limpio para trabajar:

```bash
mkdir proyecto_filosofos
cd proyecto_filosofos
nano filosofos.c
```

**- Paso 2:** Escribir el código en nano

  Copia las siguientes tres partes en orden dentro del archivo. La estrategia aplicada aquí es romper la simetría: los primeros 4 filósofos toman el tenedor izquierdo primero, pero el quinto toma primero el derecho, eliminando la posibilidad de interbloqueo.

*Parte A:* Librerías y recursos compartidos

```C
#include <stdio.h>      // Entrada y salida estándar (printf)
#include <stdlib.h>     // Números aleatorios (rand)
#include <pthread.h>    // Manejo de hilos (filósofos actuando en paralelo)
#include <semaphore.h>  // Semáforos (control de acceso a los tenedores)
#include <unistd.h>     // Pausas de tiempo (sleep)

#define TOTAL 5
sem_t tenedores[TOTAL]; // Cada semáforo representa un tenedor (1 = libre, 0 = ocupado)
```

*Parte B:* Ciclo de vida de cada filósofo
```C
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

*Parte C:* Inicialización del programa (main)
```C
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

```bash
gcc filosofos.c -o filosofos -lpthread
```

Ejecutar:
```bash
./filosofos
```
Detén la ejecución en cualquier momento presionando Ctrl + C.

## Diferentes metodos de solucion al problema de la cena de los filosofos
Cabe destacar que el codigo anterior es solamente uno de los multiples metodos que se pueden emplear para resolver este problema logico. A continuacion se explican dos metodos mas por los que se puede "resolver" dicho dilema:

**Método 2: El Portero (Semáforo Contador)**

En lugar de modificar el comportamiento de un filósofo en particular, se coloca un "portero" en la mesa que controla cuántas personas pueden intentar agarrar tenedores al mismo tiempo. Si hay 5 tenedores, el portero solo deja pasar a 4 filósofos simultáneamente. Por pura matemática, si hay 4 personas y 5 tenedores, es imposible que todos queden con un solo tenedor; al menos uno conseguirá dos.

**Que lo diferencia de los otros metodos?**

Se diferencia en que todos los filósofos son simétricos (ejecutan exactamente las mismas líneas de código), pero existe un recurso global extra (portero) que regula el acceso a la mesa.

**Pros:** Código limpio y uniforme para todos los hilos; fácil de escalar a cualquier número "N" de filósofos cambiando una constante.

**Contras:** Reduce ligeramente el paralelismo (un filósofo puede quedarse esperando en la "puerta" aunque sus dos tenedores estén libres). Tampoco garantiza evitar la inanición (starvation).

Implementación completa (filosofos_portero.c)

```C
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

#define TOTAL 5

sem_t tenedores[TOTAL];
sem_t portero; // Semáforo adicional que limita el acceso a la mesa

void* accion_filosofo(void* numero) {
    int id = *(int*)numero;
    int izq = id;
    int der = (id + 1) % TOTAL;

    while (1) {
        // 1. Pensar
        printf("Filosofo %d: Pensando...\n", id);
        sleep(rand() % 3 + 1);

        // 2. Pedir permiso al portero antes de tocar cualquier tenedor
        sem_wait(&portero);

        // 3. Tomar tenedores (orden idéntico para todos)
        sem_wait(&tenedores[izq]);
        sem_wait(&tenedores[der]);

        // 4. Comer
        printf("Filosofo %d: COMIENDO con tenedores %d y %d\n", id, izq, der);
        sleep(rand() % 3 + 1);

        // 5. Soltar tenedores y avisar al portero que sale de la mesa
        sem_post(&tenedores[izq]);
        sem_post(&tenedores[der]);
        sem_post(&portero);
    }
}

int main() {
    pthread_t filosofos[TOTAL];
    int ids[TOTAL];

    // El portero permite máximo TOTAL - 1 (4) filósofos intentando comer a la vez
    sem_init(&portero, 0, TOTAL - 1);

    for (int i = 0; i < TOTAL; i++) {
        sem_init(&tenedores[i], 0, 1);
        ids[i] = i;
    }

    for (int i = 0; i < TOTAL; i++) {
        pthread_create(&filosofos[i], NULL, accion_filosofo, &ids[i]);
    }

    for (int i = 0; i < TOTAL; i++) {
        pthread_join(filosofos[i], NULL);
    }

    return 0;
}
```
Compilar y ejecutar en Fedora:

```Bash
gcc filosofos_portero.c -o filosofos_portero -lpthread && ./filosofos_portero
```

**Método 3: Toma Atómica con Estados (Mutex + Arreglo de Estados)**

Es la solución clásica más robusta (propuesta por Tanenbaum). Aquí los tenedores dejan de ser semáforos individuales; en su lugar, se rastrea el estado de cada filósofo (PENSANDO, HAMBRIENTO, COMIENDO). Un filósofo entra a una zona protegida por un candado único (mutex), revisa si sus dos vecinos no están comiendo y, solo si ambos lados están libres, toma los dos tenedores en un solo paso atómico.

**Que lo diferencia de los otros metodos?**

Ningún filósofo toma un tenedor mientras espera el otro. O toma ambos de golpe, o no toma ninguno y se duerme hasta que un vecino le avise.

**Pros:** Máximo rendimiento y paralelismo (ningún recurso queda retenido sin usarse); elimina el interbloqueo (deadlock) y reduce drásticamente el riesgo de inanición.

**Contras:** Mayor complejidad lógica; requiere sincronizar un mutex global con semáforos privados por hilo y calcular índices de vecinos con aritmética modular.

Implementación completa (filosofos_atomico.c)

```C
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

#define TOTAL 5
#define IZQ (id + TOTAL - 1) % TOTAL  // Vecino izquierdo
#define DER (id + 1) % TOTAL          // Vecino derecho

enum { PENSANDO, HAMBRIENTO, COMIENDO } estado[TOTAL];

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER; // Protege el arreglo de estados
sem_t permiso_comer[TOTAL];                        // Semáforo privado por filósofo

// Comprueba si el filósofo 'id' puede empezar a comer
void verificar(int id) {
    if (estado[id] == HAMBRIENTO && estado[IZQ] != COMIENDO && estado[DER] != COMIENDO) {
        estado[id] = COMIENDO;
        sem_post(&permiso_comer[id]); // Le da luz verde para comer
    }
}

void tomar_tenedores(int id) {
    pthread_mutex_lock(&mutex);       // Bloquea la mesa para revisar estados sin interferencias
    estado[id] = HAMBRIENTO;
    verificar(id);                    // Intenta tomar ambos tenedores al mismo tiempo
    pthread_mutex_unlock(&mutex);     // Libera el acceso a la mesa
    sem_wait(&permiso_comer[id]);     // Si no pudo comer, espera aquí hasta que un vecino lo despierte
}

void soltar_tenedores(int id) {
    pthread_mutex_lock(&mutex);
    estado[id] = PENSANDO;
    verificar(IZQ);                   // Revisa si el vecino izquierdo tenía hambre y ya puede comer
    verificar(DER);                   // Revisa si el vecino derecho tenía hambre y ya puede comer
    pthread_mutex_unlock(&mutex);
}

void* accion_filosofo(void* numero) {
    int id = *(int*)numero;

    while (1) {
        // 1. Pensar
        printf("Filosofo %d: Pensando...\n", id);
        sleep(rand() % 3 + 1);

        // 2. Tomar ambos tenedores de forma atómica
        tomar_tenedores(id);

        // 3. Comer
        printf("Filosofo %d: COMIENDO (Vecinos %d y %d esperando)\n", id, IZQ, DER);
        sleep(rand() % 3 + 1);

        // 4. Soltar tenedores y despertar vecinos si es posible
        soltar_tenedores(id);
    }
}

int main() {
    pthread_t filosofos[TOTAL];
    int ids[TOTAL];

    for (int i = 0; i < TOTAL; i++) {
        estado[i] = PENSANDO;
        sem_init(&permiso_comer[i], 0, 0); // Inicializados en 0 (bloqueados por defecto)
        ids[i] = i;
    }

    for (int i = 0; i < TOTAL; i++) {
        pthread_create(&filosofos[i], NULL, accion_filosofo, &ids[i]);
    }

    for (int i = 0; i < TOTAL; i++) {
        pthread_join(filosofos[i], NULL);
    }

    return 0;
}
```

Compilar y ejecutar en Fedora:
```bash
gcc filosofos_atomico.c -o filosofos_atomico -lpthread && ./filosofos_atomico
```
