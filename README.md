# Paradigmas

## Ejemplo de Control de Peticiones a un Servidor con Paralelismo en Go

### Introducción
En este ejemplo, exploraremos cómo gestionar peticiones a un servidor utilizando el lenguaje de programación Go. En particular, destacaremos cómo Go implementa la concurrencia y cómo podemos aplicar el paralelismo para evitar la saturación de recursos al definir un límite en la cantidad de solicitudes simultáneas.

### Aplicando Paralelismo para Evitar Saturación
En el código proporcionado, se muestra un ejemplo de cómo aplicar el paralelismo para evitar la saturación al procesar múltiples solicitudes a un servidor. En lugar de enviar todas las solicitudes al servidor de una vez, limitamos la cantidad de solicitudes simultáneas utilizando un canal con búfer y goroutines. Esto permite que las solicitudes se procesen en paralelo, pero con un límite definido en la cantidad de solicitudes que se envían simultáneamente.

El paralelismo aquí radica en que múltiples solicitudes se procesan al mismo tiempo, lo que puede aprovechar la capacidad de múltiples núcleos de CPU para acelerar el procesamiento. Al mismo tiempo, la concurrencia se logra mediante el uso de goroutines y canales para gestionar el flujo de trabajo de manera eficiente.




### Código
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

#### Playground
```
https://go.dev/play/
```