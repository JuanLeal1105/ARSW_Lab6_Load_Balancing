# **Escalamiento en Azure con Máquinas Virtuales, Scale Sets y Service Plans**
 
**Integrantes:**

Juan Carlos Leal Cruz

## **Conexión con la VM e Instalación de paquetes**
1. Se realiza la conexión mediante ssh usando la `.pem` generada por Azure.
![alt text](images/evidences/ConnectionVM.png)

2. Luego se debe proceder a ejecutar los siguientes comandos para instalar `node`
```bash
sudo apt update
sudo apt install nodejs

sudo apt install npm     # In case npm was not installed along with nodejs
```

3. Se realiza la instalación de paquetes de los archivos `.js` y se ejecuta la app
```bash
npm install
node FibonacciApp.js
```
![alt text](images/evidences/ExecutionNpm.png)

4. Se hace ejecución de una pequeña prueba luego de la configuración pedida del inbound rule
![alt text](images/evidences/TestFibo6.png)

## **Parte 1 - Escalabilidad Vertical**
### Evidencias
 
#### **Tabla de tiempos con B1ls (1 vCPU, 0.5 GiB RAM):**
![alt text](images/evidences/ValuesNumbers.png)
| N (Fibonacci) | Tiempo de respuesta (ms) | Tiempo de respuesta (s) |
|---|---|---|
| 1000000 | 12795 | 12.80 |
| 1010000 | 13049 | 13.05 |
| 1020000 | 13624 | 13.62 |
| 1030000 | 13564 | 13.56 |
| 1040000 | 13740 | 13.74 |
| 1050000 | 14166 | 14.17 |
| 1060000 | 14606 | 14.61 |
| 1070000 | 15010 | 15.01 |
| 1080000 | 15331 | 15.33 |
| 1090000 | 15860 | 15.86 |
 
**Consumo de CPU — B1ls:**
![alt text](images/evidences/CPU_Usage.png)
 
**Resultado Newman — B1ls (2 ejecuciones paralelas):**
![alt text](images/evidences/Instancia1.png)
![alt text](images/evidences/Instancia2.png)
 
| Métrica | Instancia 1 | Instancia 2 |
|---|---|---|
| Iteraciones ejecutadas | 10 | 10 |
| Requests exitosos | 5/10 | 6/10 |
| Requests fallidos | 5 (ECONNRESET) | 4 (ECONNRESET) |
| Tiempo promedio de respuesta | 17.8s | 15.9s |
| Tiempo mínimo | 12.8s | 12.8s |
| Tiempo máximo | 24.9s | 24.8s |
| Desviación estándar | 5.8s | 5.1s |
| Duración total | 2m 20.1s | 2m 20.4s |
| Datos recibidos | 1.05 MB | 1.25 MB |
 
#### **Tabla de tiempos con B2ms (2 vCPU, 8 GiB RAM):**
![alt text](images/evidences/ValuesNumbers2.png)
| N (Fibonacci) | Tiempo B1ls (ms) | Tiempo B1ls (s) | Tiempo B2ms (ms) | Tiempo B2ms (s) |
|---|---|---|---|---|
| 1000000 | 12795 | 12.80 | 8536 | 8.54 |
| 1010000 | 13049 | 13.05 | 8715 | 8.72 |
| 1020000 | 13624 | 13.62 | 8736 | 8.74 |
| 1030000 | 13564 | 13.56 | 8791 | 8.79 |
| 1040000 | 13740 | 13.74 | 8993 | 8.99 |
| 1050000 | 14166 | 14.17 | 9222 | 9.22 |
| 1060000 | 14606 | 14.61 | 9216 | 9.22 |
| 1070000 | 15010 | 15.01 | 9515 | 9.52 |
| 1080000 | 15331 | 15.33 | 9732 | 9.73 |
| 1090000 | 15860 | 15.86 | 9887 | 9.89 |
 
**Consumo de CPU — B2ms:**
![alt text](images/evidences/CPU_Usage2.png)
 
**Resultado Newman — B2ms (2 ejecuciones paralelas):** 
| Métrica | Instancia 1 | Instancia 2|
|---|---|---|
| Iteraciones ejecutadas | 10 | 10 |
| Requests exitosos | 7/10 | 7/10 |
| Requests fallidos | 3 (ECONNRESET) | 3 (ECONNRESET) |
| Tiempo promedio de respuesta | 14.2s | 14.5s |
| Tiempo mínimo | 11.5s | 11.5s |
| Tiempo máximo | 22.1s | 22.4s |
| Duración total | 2m 5s | 2m 8s |
 
> La mejora se explica porque con 2 vCPUs el SO gestiona mejor los procesos concurrentes, reduciendo la cantidad de conexiones reseteadas. Sin embargo, Node.js sigue sin paralelizar el cálculo, por lo que la mejora no es radical.

