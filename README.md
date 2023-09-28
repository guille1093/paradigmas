# paradigmas

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	requestIDs := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20}
	concurrencyLimit := 3

	//Se crea un canal llamado requestChannel con un búfer de tamaño concurrencyLimit para limitar el número de solicitudes que pueden procesarse en paralelo.
	requestChannel := make(chan int, concurrencyLimit)

	// Utilizamos WaitGroup para esperar a que todas las goroutines finalicen.
	var wg sync.WaitGroup

	// Iniciar las goroutines de procesamiento.
	for i := 0; i < concurrencyLimit; i++ {
		wg.Add(1)
		go processRequests(&wg, requestChannel)
	}

	// Medir el tiempo de inicio.
	startTime := time.Now()

	// Enviar solicitudes al canal.
	sendRequests(requestIDs, requestChannel)

	// Cerrar el canal después de enviar todas las solicitudes.
	close(requestChannel)

	// Esperar a que todas las goroutines de procesamiento finalicen.
	wg.Wait()

	// Medir el tiempo de finalización.
	endTime := time.Now()

	// Calcular la duración total y mostrarla.
	duration := endTime.Sub(startTime)
	fmt.Println("Tiempo total en ejecución paralela:", duration.Seconds(), "segundos")

	// Calcular el tiempo en ejecución secuencial.
	sequentialStartTime := time.Now()
	sequentialProcessRequests(requestIDs)
	sequentialEndTime := time.Now()
	sequentialDuration := sequentialEndTime.Sub(sequentialStartTime)
	fmt.Println("Tiempo total en ejecución secuencial:", sequentialDuration.Seconds(), "segundos")
}

// Funciones auxiliares

func sendRequests(requestIDs []int, requestChannel chan<- int) {
	// Función para enviar solicitudes al canal.
	for _, ID := range requestIDs {
		requestChannel <- ID // Enviar ID al canal
	}
}

func processRequests(wg *sync.WaitGroup, requestChannel <-chan int) {
	// Función para procesar solicitudes desde el canal.
	defer wg.Done() // Marcar la goroutine como finalizada al salir.
	for ID := range requestChannel {
		makeRequest(ID)
	}
}

func makeRequest(ID int) {
	time.Sleep(1 * time.Second)
	fmt.Println("Solicitud procesada:", ID)
}

func sequentialProcessRequests(requestIDs []int) {
	// Función para procesar solicitudes secuencialmente.
	for _, ID := range requestIDs {
		makeRequest(ID)
	}
}
```
