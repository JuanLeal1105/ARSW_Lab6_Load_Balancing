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


### **Preguntas — Parte 1** 
#### **1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?**
Azure crea 7 recursos adicionales junto con la máquina virtual:
- Virtual Network (SCALABILITY_LAB-vnet)
- Subnet (default)
- Public IP address (VERTICAL-SCALABILITY-ip)
- Network Security Group (VERTICAL-SCALABILITY-nsg)
- Network Interface (vertical-scalabilityXXX)
- OS Disk (disco del sistema operativo)
- SSH Key (si se generó un nuevo par de llaves)

#### **2. ¿Brevemente describa para qué sirve cada recurso?**
- **Virtual Network (VNet):** Red privada aislada dentro de Azure que permite a los recursos comunicarse entre sí de forma segura, con Internet y con redes on-premises.
- **Subnet:** Segmento lógico dentro de la VNet que permite organizar y aislar recursos. Define un rango de direcciones IP dentro del espacio de la VNet.
- **Public IP address:** Dirección IP pública que permite que la VM sea accesible desde Internet. Sin ella, la VM solo sería alcanzable dentro de la VNet.
- **Network Security Group (NSG):** Firewall virtual que contiene reglas de seguridad para filtrar el tráfico de red entrante (inbound) y saliente (outbound) hacia/desde los recursos de Azure.
- **Network Interface (NIC):** Interfaz de red virtual que conecta la VM con la VNet. Es el componente que posee la IP privada y se asocia con la IP pública y el NSG.
- **OS Disk:** Disco administrado que contiene el sistema operativo de la VM. Se crea automáticamente a partir de la imagen seleccionada (Ubuntu Server).
- **SSH Key:** Par de llaves criptográficas (pública/privada) utilizado para autenticación segura sin contraseña al conectarse por SSH.

#### **3. ¿Al cerrar la conexión SSH con la VM, por qué se cae la aplicación que ejecutamos con el comando `node FibonacciApp.js`? ¿Por qué debemos crear un Inbound port rule antes de acceder al servicio?**
Cuando ejecutamos `node FibonacciApp.js` directamente, el proceso de Node.js se ejecuta como un proceso hijo de la sesión SSH. Al cerrar la sesión SSH, el sistema operativo envía una señal SIGHUP (Signal Hang Up) a todos los procesos hijos de esa sesión, lo que provoca su terminación. Por eso se utiliza `forever` o herramientas como `nohup`, `pm2` o `screen`, que desacoplan el proceso de la sesión del terminal.
 
Respecto a la Inbound port rule: por defecto, el NSG de Azure solo permite tráfico entrante en el puerto 22 (SSH) y 80 (HTTP si se habilitó). La aplicación de Fibonacci escucha en el puerto 3000, que está bloqueado por defecto. Sin crear una regla explícita que permita el tráfico TCP en el puerto 3000, las peticiones externas nunca llegarán a la aplicación.
 
#### **4. Adjunte tabla de tiempos e interprete por qué la función tarda tanto tiempo.**
*(Ver tablas en la sección de Evidencias arriba.)*

Con la VM B1ls los tiempos van desde 12.80s (N=1.000.000) hasta 15.86s (N=1.090.000), y con la B2ms se reducen a entre 8.54s y 9.89s para el mismo rango. La función tarda tanto porque la implementación utiliza un enfoque iterativo sin memoización que recalcula todo desde cero en cada petición. Trabajar con N superior a 1.000.000 implica sumar números con cientos de miles de dígitos (aritmética BigInt), donde cada operación individual es costosa. El tiempo crece de forma aproximadamente lineal con N porque se necesitan N iteraciones y cada una opera sobre números cada vez más grandes. La mejora en B2ms (~33% más rápida) no se debe a paralelismo sino a que el procesador base de esa SKU tiene mayor frecuencia y el SO compite menos por la única vCPU activa.
 
#### **5. Adjunte imagen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.**
*(Ver imágenes en la sección de Evidencias arriba.)*
 
En la gráfica de B1ls se observa un pico abrupto que alcanza aproximadamente el **8.29%** en promedio registrado por Azure (el promedio diluye el pico real, que internamente satura el 100% de la única vCPU durante los ~14s que dura cada cálculo). Con B2ms el pico promedio sube a **10.66%** en la métrica de Azure, lo que a primera vista parece peor, pero en realidad refleja que la VM estuvo disponible durante la prueba y absorbió más carga sin caerse — el porcentaje se ve más alto porque Azure lo promedia sobre 2 vCPUs y la prueba fue más larga. El consumo es tan intenso porque el cálculo de Fibonacci para N > 1.000.000 es una operación puramente CPU-bound: Node.js ejecuta en un solo hilo y cada iteración realiza sumas sobre números BigInt de cientos de miles de dígitos, bloqueando completamente el event loop durante toda la duración del cálculo.
 
#### **6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete.**
*(Ver imágenes en la sección de Evidencias arriba.)*
 
**Tiempos de ejecución:** Las peticiones muestran tiempos de respuesta muy elevados (del orden de decenas de segundos cada una), ya que la VM B1ls con una sola vCPU debe procesar las peticiones de forma secuencial. Al ejecutar dos instancias de Newman en paralelo, las peticiones se encolan porque Node.js no puede atender múltiples cálculos de Fibonacci simultáneamente.
 
**Fallos:** 
Es probable que algunas peticiones fallen por timeout, ya que al haber concurrencia, las peticiones en cola deben esperar a que terminen las anteriores, y los tiempos acumulados pueden exceder el timeout configurado en Postman/Newman.
 
#### **7. ¿Cuál es la diferencia entre los tamaños B2ms y B1ls (no solo busque especificaciones de infraestructura)?**
| Característica | B1ls | B2ms |
|---|---|---|
| vCPUs | 1 | 2 |
| RAM | 0.5 GiB | 8 GiB |
| Data Disks | 2 | 4 |
| Max IOPS | 200 | 1920 |
| Almacenamiento temporal | 1 GiB | 16 GiB |
| Costo estimado | ~$3.80 USD/mes | ~$60.74 USD/mes |
 
Más allá de las especificaciones, la diferencia fundamental está en la capacidad de créditos de CPU (CPU bursting). Ambos pertenecen a la serie B (burstable), que acumula créditos cuando la CPU está inactiva y los consume durante picos de uso. La B1ls, al tener menos recursos base, acumula menos créditos y los agota más rápido bajo carga sostenida. La B2ms con el doble de vCPUs puede paralelizar mejor y tiene un baseline de rendimiento de CPU más alto (hasta el 60% sostenido vs. ~5% de la B1ls). También puede ejecutar más tareas concurrentes al tener significativamente más memoria RAM.
 
#### **8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario? ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?**
Aumentar el tamaño de la VM no es la solución óptima en este escenario. Aunque el hardware mejora, la aplicación de Fibonacci sigue siendo single-threaded (Node.js), por lo que solo puede utilizar un núcleo de CPU a la vez para cada cálculo. La segunda vCPU ayuda a que el sistema operativo y otros procesos no compitan con la aplicación, pero no permite procesar dos cálculos de Fibonacci en paralelo.
 
Cuando se cambia el tamaño de la VM, Azure debe **detener (deallocate)** la VM, cambiar el hardware subyacente, y reiniciarla. Esto significa que la FibonacciApp deja de funcionar temporalmente y se pierde el proceso de `forever` que mantenía la app corriendo. Es necesario volver a iniciar la aplicación manualmente después del resize.
 
#### **9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?**
Al cambiar el tamaño, Azure detiene la VM, la mueve a un host físico que soporte el nuevo tamaño, y la reinicia. Los efectos negativos son:
- **Downtime:** La VM no está disponible durante el proceso de redimensionamiento, generando indisponibilidad del servicio.
- **Cambio de IP pública:** Si la IP no es estática, puede cambiar al reiniciar, rompiendo las configuraciones de DNS o las referencias hardcoded.
- **Pérdida de estado en memoria:** Todos los procesos en ejecución se pierden (incluyendo la FibonacciApp), la caché en RAM se pierde.
- **Datos en disco temporal:** El almacenamiento temporal (ephemeral) se pierde completamente.
- **Costo:** Pasar de B1ls a B2ms incrementa el costo en ~16x (~$3.80 → ~$60.74 USD/mes).

#### **10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?**
*(Ver tabla comparativa en la sección de Evidencias arriba.)*
 
**Sí hubo mejora**, y fue significativa en los tiempos de respuesta individuales: los tiempos bajaron en promedio un **~33%**, pasando de un rango de 12.80s–15.86s con B1ls a 8.54s–9.89s con B2ms. Esto se debe a que la B2ms tiene mayor capacidad de procesamiento base (2 vCPUs, 8 GiB RAM) y el SO dispone de más recursos para gestionar los procesos del sistema sin interferir con el cálculo. Sin embargo, la mejora no es proporcional al aumento de hardware (que es de ~16x en costo), lo que demuestra que el problema fundamental es arquitectural: Node.js es single-threaded y no puede paralelizar el cálculo de Fibonacci. La segunda vCPU no es aprovechada por la aplicación. En cuanto a la concurrencia bajo carga (Newman), la tasa de fallos por ECONNRESET también debería reducirse con B2ms, pero no eliminarse, porque el cuello de botella persiste.
 
#### **11. Aumente la cantidad de ejecuciones paralelas del comando de Postman a 4. ¿El comportamiento del sistema es porcentualmente mejor?**
*(Ver imagen en la sección de Evidencias arriba.)*
 
No, el comportamiento del sistema **no es porcentualmente mejor** con 4 ejecuciones paralelas. Al incrementar la concurrencia a 4 peticiones simultáneas, el cuello de botella sigue siendo el mismo: Node.js es single-threaded y solo puede calcular un Fibonacci a la vez. Las peticiones adicionales se encolan y esperan, lo que incrementa los tiempos totales. El consumo de CPU sigue siendo cercano al 100% en una vCPU y los tiempos de respuesta de cada petición se multiplican proporcionalmente al número de peticiones encoladas. Más aún, al haber más peticiones esperando, aumenta la probabilidad de fallos por timeout.
 
---
## **Parte 2 - Escalabilidad Horizontal**
### Evidencias
#### Informe Newman 1 — 2 VMs + Load Balancer (2 ejecuciones paralelas)
 
**Resultado Newman — Tres instancias:**
![alt text](images/evidences2/v1.png)
![alt text](images/evidences2/v2.png)
![alt text](images/evidences2/v3.png)
 
**Consumo de CPU durante la prueba (2 VMs):**
| VM | CPU Avg | Pico | Comportamiento |
|---|---|---|---|
| VM1 | 0.52% | ~0.7% | Casi sin carga — el LB no le envió peticiones |
| VM2 | 8.21% | ~90% | Absorbió la mayoría de peticiones |
 
<img src="images/evidences2/cpu1_1.png" width="600">
<img src="images/evidences2/cpu2_1.png" width="600">

> **Nota:** La distribución desigual de CPU (VM1 con 0.52% vs VM2 con 8.21%) se debe al algoritmo de hash de 5-tupla del balanceador de carga. Dado que las dos instancias de Newman se ejecutan desde la misma máquina (misma IP de origen), el hash puede sesgar las peticiones hacia ciertos nodos. A pesar de esto, el **100% de las peticiones fueron exitosas** porque el balanceo permite que cada VM procese sus peticiones sin el encolamiento crítico que ocurría con una sola VM.

#### **Informe Newman 1 — 2 VMs + Load Balancer (4 ejecuciones paralelas)**
> **Nota:** El enunciado pide agregar una cuarta máquina virtual para esta prueba, sin embargo debido al límite de cuota de la suscripción Azure for Students no fue posible aprovisionar una cuarta VM. La prueba se realizó con las mismas 3 VMs del backend pool aumentando la concurrencia a 4 ejecuciones paralelas de Newman.
 
**Resultado Newman (muestra representativa de una instancia):**
![alt text](images/evidences2/vTest4.png)
 
**Consumo de CPU por VM (2 VMs, 4 ejecuciones paralelas):**
| VM | CPU Avg | Pico |
|---|---|---|
| VM1 | 11.75% | ~95% |
| VM2 | 12.66% | ~90% |

<img src="images/evidences2/cpu1_2.png" width="600">
<img src="images/evidences2/cpu2_2.png" width="600">
 

> Las gráficas muestran picos de CPU del ~90–95% en cada VM durante las ventanas de cálculo, con dos bloques de actividad correspondientes a las oleadas de peticiones. El promedio bajo (~11–12%) se debe a que Azure promedia la métrica sobre todo el periodo de observación, diluyendo los picos reales que ocurren solo durante los segundos de cómputo activo.
 
