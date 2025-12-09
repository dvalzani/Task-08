### Task 08

```c

#include <stdio.h>      // Include per printf
#include <stdlib.h>     // Include per malloc/free
#include <math.h>       // Include per ceil()
#include <stdbool.h>    // Include per tipo bool (true/false)


// -------------------------------------------------------------
// Funzione SERIAL: d[i] = a*x[i] + y[i]
// -------------------------------------------------------------
// "void" significa che la funzione NON restituisce alcun valore.
// Tu le passi i vettori, e lei li modifica direttamente.
void daxpy_serial(double *d, const double *x, const double *y, double a, int n) {

    // classico ciclo for da 0 a n-1
    for (int i = 0; i < n; i++) {
        d[i] = a * x[i] + y[i];   // operazione daxpy
    }
}



// -------------------------------------------------------------
// Funzione CHUNKED: divide il lavoro in blocchi
// -------------------------------------------------------------
// double *d  = vettore di output
// const double *x, *y = vettori di input
// a = scalare
// n = dimensione dei vettori
// chunk_size = grandezza dei blocchi
// partial_chunk_sum = array che conterrà la somma dei singoli chunk
void daxpy_chunked(double *d, const double *x, const double *y, double a,
                   int n, int chunk_size, double *partial_chunk_sum) {

    // numero di chunk = ceil(n / chunk_size)
    // esempio: n=100, chunk=8  → 100/8 = 12.5 → ceil → 13
    int number_of_chunks = (int)ceil((double)n / chunk_size);

    // ciclo su ogni chunk
    for (int c = 0; c < number_of_chunks; c++) {

        // posizione iniziale del chunk
        int start = c * chunk_size;

        // posizione finale del chunk
        int end = (c + 1) * chunk_size;

        // se l'ultimo chunk va oltre n, lo tagliamo
        if (end > n)
            end = n;

        // somma locale del chunk
        double local_sum = 0.0;

        // ciclo interno sugli elementi del chunk
        for (int i = start; i < end; i++) {

            d[i] = a * x[i] + y[i];   // calcolo daxpy
            local_sum += d[i];        // accumulo la somma del chunk
        }

        // scrivo la somma locale nell'array dei risultati
        partial_chunk_sum[c] = local_sum;
    }
}



// -------------------------------------------------------------
// MAIN — dove il programma parte
// -------------------------------------------------------------
int main() {

    int n = 100;            // numero di elementi del vettore
    int chunk_size = 8;     // dimensione del chunk
    double a = 3.0;         // scalare

    // ---------------------------------------------------------
    // ALLOCAZIONE DEI VETTORI
    // malloc crea spazio in RAM per n double
    // ---------------------------------------------------------
    double *x = malloc(n * sizeof(double));
    double *y = malloc(n * sizeof(double));
    double *d_serial = malloc(n * sizeof(double));
    double *d_chunked = malloc(n * sizeof(double));

    // numero di chunk calcolato qui per potere allocare partial_sum
    int num_chunks = (int)ceil((double)n / chunk_size);

    double *partial_sums = malloc(num_chunks * sizeof(double));

    // ---------------------------------------------------------
    // INIZIALIZZO x e y
    // ---------------------------------------------------------
    for (int i = 0; i < n; i++) {
        x[i] = 0.1;
        y[i] = 7.1;
    }

    // ---------------------------------------------------------
    // Eseguo la versione seriale e quella chunked
    // ---------------------------------------------------------
    daxpy_serial(d_serial, x, y, a, n);
    daxpy_chunked(d_chunked, x, y, a, n, chunk_size, partial_sums);


    // ---------------------------------------------------------
    // CONTROLLO CHE I DUE VETTORI SIANO UGUALI
    // ---------------------------------------------------------
    bool ok = true;
    for (int i = 0; i < n; i++) {
        if (fabs(d_serial[i] - d_chunked[i]) > 1e-12) {
            ok = false;
        }
    }

    // ---------------------------------------------------------
    // SOMMA DEI CHUNK
    // ---------------------------------------------------------
    double sum_chunks = 0.0;
    for (int i = 0; i < num_chunks; i++) {
        sum_chunks += partial_sums[i];
    }

    // ---------------------------------------------------------
    // SOMMA SERIALE
    // ---------------------------------------------------------
    double sum_serial = 0.0;
    for (int i = 0; i < n; i++) {
        sum_serial += d_serial[i];
    }

    // ---------------------------------------------------------
    // STAMPO RISULTATI
    // ---------------------------------------------------------
    printf("Check vettori d: %s\n", ok ? "OK" : "FAIL");
    printf("Somma parziale su chunk = %f\n", sum_chunks);
    printf("Somma seriale completa =  %f\n", sum_serial);

    // ---------------------------------------------------------
    // LIBERO LA MEMORIA
    // ---------------------------------------------------------
    free(x);
    free(y);
    free(d_serial);
    free(d_chunked);
    free(partial_sums);

    return 0;   // fine corretta del programma
}

```

## Output

```

[root@be0669f7bc73 task08]# ./daxpy_split
Check vettori d: OK
Somma parziale su chunk = 740.000000
Somma seriale completa =  740.000000

```
