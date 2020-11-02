### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

## Integrantes

* Angi Paola Jimenez Pira
* Sergio Alejandro Bohorquez Alzate

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/en-us/free/search/?&ef_id=Cj0KCQiA2ITuBRDkARIsAMK9Q7MuvuTqIfK15LWfaM7bLL_QsBbC5XhJJezUbcfx-qAnfPjH568chTMaAkAsEALw_wcB:G:s&OCID=AID2000068_SEM_alOkB9ZE&MarinID=alOkB9ZE_368060503322_%2Bazure_b_c__79187603991_kwd-23159435208&lnkd=Google_Azure_Brand&dclid=CjgKEAiA2ITuBRDchty8lqPlzS4SJAC3x4k1mAxU7XNhWdOSESfffUnMNjLWcAIuikQnj3C4U8xRG_D_BwE). Al hacerlo usted contará con $200 USD para gastar durante 1 mes.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**

Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

    ![Imágen 1](images/part1/part1-vm-basic-config.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM.

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    `npm install forever -g`

    `forever start FibinacciApp.js`

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

    ![](images/part1/part1-vm-3000InboudRule.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000:
        ![Imágen](images/1000000.png)
    * 1010000:
        ![Imágen](images/1010000.png)
    * 1020000:
        ![Imágen](images/1020000.png)
    * 1030000:
        ![Imágen](images/1030000.png)
    * 1040000:
        ![Imágen](images/1040000.png)
    * 1050000:
        ![Imágen](images/1050000.png)
    * 1060000:
        ![Imágen](images/1060000.png)
    * 1070000:
        ![Imágen](images/1070000.png)
    * 1080000:
        ![Imágen](images/1080000.png)
    * 1090000:
        ![Imágen](images/1090000.png)

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

    ![Imágen 2](images/part1/part1-vm-cpu.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

    ![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
    
     Al aumentar el tamaño y realizar las mismas peticiones el porcentaje de consumo de CPU disminuyó pero el tiempo de respuesta se mantuvo constante, por lo que se puede inferir la estrategia de escalamiento vertical implementada no cumple el objetivo, esto puede ser porque el programa no aprovecha los recusos de CPU de la maquina virtual.
     
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. **¿Cuántos y cuáles recursos crea Azure junto con la VM?**

    Azure crea 6 recursos junto con la máquina virtual, estos son:
    * Virtual Network
    * Storage Account
    * Public IP Address
    * Network Security Group
    * Network Interface
    * Disk
    
2. **¿Brevemente describa para qué sirve cada recurso?**
    
    - **Virtual Network** tiene como objetivo brindar comunicación entre recursos de azure como Azure Virtual Machines, es similar a una red tradicional, sin embargo ofrece beneficios adicionales de Azure como escalabilidad, aislamiento y disponibilidad.
    
    - **Storage Account** contiene todos los objetos de datos de Azure Storage: blobs, archivos, colas, tablas y discos. Proporciona un espacio de nombres único para sus datos de Azure Storage al que se puede acceder desde cualquier lugar del mundo a través de HTTP o HTTPS, los datos de la cuenta de almacenamiento de Azure son duraderos y de alta disponibilidad, seguros y escalables de forma masiva.

    - **Public IP Address** permite que los recursos de azure se comuniquen con internet y con servicios públicos de azure, esta dirección es asignada a un recurso hasta que este se elimine. Un recurso de dirección IP pública se puede asociar con: Interfaces de red de máquinas virtuales, balanceadores de carga orientados a Internet, pasarelas VPN, pasarelas de aplicación, cortafuegos Azure.
    
    - **Network Security Group** permite filtrar el tráfico hacia y desde los recursos en una red virtual de Azure, un grupo de seguridad permite definir reglas de entrada y/o salida que permitan o denieguen el tráfico de red entrante o saliente de varios tipos de recursos de Azure.
    
    - **Network Interface** permite que una máquina virtual Azure se comunique con el Internet y con recursos locales.
    
    - **Disk** Almacenamiento de la máquina virtual Azure.
    
3. **¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?**

    Cuando nos conectamos a la máquina virtual mediante SSH, se inicia un proceso para este servicio y todos los comandos ejecutados a partir de ahi crearán hijos de dicho proceso, que terminarán en cuanto se finalice la conexión mediante SSH.

    Se debe crear una regla de entrada en el puerto 3000 para exponer el servicio de FibonacciApp en internet y permitir el acceso externo esta regla puede ser TCP/UDP.

4. **Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.**
    
    | N       | B1ls(s) | B2ms(s) |
    |---------|---------|---------|
    | 1000000 | 20.68   | 20.03   |
    | 1010000 | 21.38   | 21.00   |
    | 1020000 | 21.39   | 20.95   |
    | 1030000 | 22.13   | 21.17   |
    | 1040000 | 22.40   | 22.02   |
    | 1050000 | 22.91   | 22.16   |
    | 1060000 | 23.71   | 22.61   |
    | 1070000 | 24.12   | 23.12   |
    | 1080000 | 24.44   | 23.36   |
    | 1090000 | 24.84   | 24.04   |

    La implementación de la función de Fibonacci no aprovecha bien los recursos del sistema al estar implementada iterativamente y no usar más hilos, se repiten cálculos para hallar el resultado de cada iteración que podrían ser almacenados en memoria.

5. **Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.**

   ![Imágen](images/cpu2.png)
   
   Cada petición consume gran parte de recursos de la cpu debido a que se realizan múltiples iteraciones en las que se realizan cálculos innecesarios, además no se implementa concurrencia lo que hace que se consuman más recursos y el tiempo de respuesta sea extenso.   
   
6. **Adjunte la imagen del resumen de la ejecución de Postman. Interprete:**

   **Resumen B1ls**

   ![Resumen1](https://media.discordapp.net/attachments/352624122301513730/771856821605761071/unknown.png)
   
   El tiempo promedio de ejecución para cada petición fue de 27.4s y se recibió un total de 1.2MB
   
   Al realizar las peticiones concurrentes se evidenciaron 4 fallos en la conexión debido a que el servidor no soporta concurrencia.
   
   ![Resumen2](https://media.discordapp.net/attachments/352624122301513730/771088449821868032/unknown.png)
   
   El tiempo promedio de ejecución para cada petición fue de 28.3s y se recibió un total de 1MB
   
   Al realizar las peticiones concurrentes se evidenciaron 5 fallos en la conexión debido a que el servidor no soporta concurrencia.
   
7. **¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?**

    La máquina B1ls tiene menos capacidad que la máquina B2ms, es mucho más económica que la B2ms y solo está disponible para linux a diferencia de la B2ms.

    | Name | vCpu (núcleos) | Ram | Discos de datos | E/S máxima por segundo | Almacenamiento temporal (GiB) | Costo/Mes ($) |
    |------|----------------|-----|-----------------|------------------------|-------------------------------|---------------|
    | B2ms | 2              | 8   | 4               | 1920                   | 16                            | 72.85         |
    | B1ls | 1              | 0.5 | 2               | 160                    | 4                             | 4.56          |

    Ambas máquinas son de uso general, y proporcionan un uso equilibrado de la CPU, son utilizadas para entornos de desarrollo y pruebas, por lo general el tráfico soportado
por estas es bajo/medio.

8. **¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?**

   Aumentar el tamaño de la máquina puede significar un consumo menor de recursos de cpu, sin embargo, no se observa mejora en los tiempos de respuesta de las peticiones ni en la capacidad de respuesta concurrente del sistema (algunas peticiones aún fallan). Si se desea mejorar los tiempos de respuesta se debe realizar una mejor implementación de la aplicación FibonacciApp.

    Cuando cambiamos el tamaño de la máquina virtual es necesario reiniciarla, por lo tanto se pierde disponibilidad de la aplicación FibonacciApp ya que esta deja de funcionar mientras se reinicia.

9. **¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?**

    Es necesario reiniciar la máquina, por lo tanto la infraestructura no estará disponible durante algunos minutos, lo cual implica que todas las peticiones entrantes durante estos minutos serán ignoradas. Si el sistema es consultado frecuentemente podría significar una perdida ya sea económica o de integridad de algunas transacciones realizadas por los clientes.    

10. **¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?**

    Si hubo mejora en el uso de cpu ya que el consumo fue del 13% en promedio comparado con el de la otra máquina que fue del 30%, esto se debe a que se disponen de más recursos para hacer los cálculos, sin embargo el tiempo de respuesta no mejoró considerablemente esto se debe a la implementación propia del programa.

11. **Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?**
    
    ![](/images/tabla3.png)
    
    El comportamiento del sistema no mejoró, el porcentaje de peticiones que fallan sigue siendo el 40%. El tiempo de respuesta no mejora significativamente, esto puede deberse a que el tamaño B2ms no mejora mucho los cores de cpu con respecto al tamaño B1ls.

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

    ![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

    ![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

    ![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

    ![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

    ![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

    ![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

    ![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

    ![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

    ![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

    ```
    git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
    source /home/vm1/.bashrc
    nvm install node

    cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
    npm install

    npm install forever -g
    forever start FibonacciApp.js
    ```

    Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

    ```
    http://52.155.223.248/
    http://52.155.223.248/fibonacci/1
    ```
   
  *  http://52.155.223.248/
  
     ![](images/parte2-1.png)
  
  * http://52.155.223.248/fibonacci/1
  
     ![](images/parte2-2.png)
     

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

   ## Informe
   
      * Resumen newman 1
      
         ![Resumen1](https://media.discordapp.net/attachments/352624122301513730/771088449821868032/unknown.png)
      
      * Resumen newman 2

         ![Resumen1](images/tabla2.png)
         
      * Tabla Comparativa Escalabilidad (Horizontal vs Vertical)
         
         | Escalamiento | Region          | SO                 | Nivel    | Instancia | Cantidad de vm | Horas (h) | Tiempo promedio (s) | Costo /Mes ($) | Peticiones éxitosas |
         |--------------|-----------------|--------------------|----------|-----------|----------------|-----------|---------------------|----------------|---------------------|
         | Vertical     | EEUU            | Linux Ubuntu 18.04 | Estándar | B2ms      | 1              | 730       | 25.3                | 66.48          | 6                   |
         | Horizontal   | Norte de Europa | Linux Ubuntu 18.04 | Estándar | B1ls      | 4              | 730       | 26.6                | 16.69          | 10                  |
         
         *(Costo calculado con Azure Calculator)*
      
      **Conclusiones**

      * El costo total de la infraestructura de escalamiento vertical es de 66.48 dolares mensuales suponiendo una disponibilidad de 730 horas, el costo de la arquitectura de escalamiento horizontal es de 39.94 dolares (16.69 por las máquinas virtuales y 23.25 por el balanceador de carga estándar) con la misma disponibilidad en horas de la infraestructura vertical.

      * La diferencia de tiempo de respuesta no es muy significativa.

      * Podemos observar que la infraestructura de la escalabilidad vertical no responde todas las peticiones exitosamente a diferencia de la de escalabilidad horizontal, esto se debe a que las replicas de los nodos permiten realizar peticiones concurrentes a diferencia de utilizar una sola máquina para computar todas las peticiones.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```
   
   * VM1
   
     ![](images/cpuvm1.png)
   
   * VM2
    
     ![](images/cpuvm2.png)
   
   * VM3
    
     ![](images/cpuvm3.png)
   
   * VM4 
    
     ![](images/cpuvm4.png)
   
**Preguntas**

* **¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?**

    * **Balanceador de carga público:** Se utilizan para equilibrar la carga proveniente de internet a las máquinas virtuales, la dirección ip pública y el puerto del tráfico entrante son asignados a la dirección privada.
        
        ![Public loadbalancer](https://media.discordapp.net/attachments/352624122301513730/772862906710097990/unknown.png)
        
     * **Balanceador de carga interno (privado):** Se utiliza para equilibrar la carga del tráfico dentro de una red privada, solo utilizan ips privadas en la interfaz

        ![Private loadbalancer](https://media.discordapp.net/attachments/352624122301513730/772863099367063592/unknown.png)
        
    * **SKU (Stock Keeping Unit):** Las unidades de mantenimiento de existencias son un código único asignado a un servicio en azure existen y representan la posibilidad para comprar existestencias. Existen varios tipos de sku entre estos tenemos:
        * **Estándar:** Son productos estándar y se pueden vender individualmente o en paquetes conjuntos o colecciones.
        * **Componente:**  Son productos incluidos en los paquetes, ensamblajes y colecciones, no pueden venderse individualmente.
        * **Ensamblaje:** Se refieren a productos que se deben ensamblar antes del envío, todos los SKU deben estar dentro de la misma instalación esta debe ser local o de un proveedor Dropship/JIT/3PL.
        * **Paquete:** No es necesario ensamblar antes del envío, debe haber disponibilidad completa y diferentes fuentes de cumplimiento.
        * **Colección:**  Son asociados a productos de marketing y solo se pueden vender SKU asosciadas.
        * **Virtual:** Son podructos que no necesitan instalación física, y no requieren un nivel de inventario.
    
            Para los balanceadores de carga se tienen 2 SKU Basic y Standard, estos definen la colección de productos a adquirir.
            
        El balanceador de carga necesita una **IP pública** para poder cumplir con su función que es capturar el tráfico y distribuirlo correctamente entre los diferentes nodos de la red.
        
* **¿Cuál es el propósito del Backend Pool?**
        
    **Backend Pool** es un componente del balacenador de carga que define el grupo de recursos que brindarán tráfico para una  Load Balancing Rule determinada, es un grupo de máquinas virtuales o instancias que atienden las solicitudes entrantes. Para escalar de manera rentable y satisfacer grandes volúmenes de tráfico entrante, generalmente se recomienda agregar más instancias a este grupo.

* **¿Cuál es el propósito del Health Probe?**
    
    Cuando se configura un nuevo balanceador de carga se crea una **Health probe** que usará el balanceador para determinar si las instancias dentro del *Backend Pool* están en buen estado, si una instancia falla un determinado número de veces entonces esta dejará de dirigir tráfico hacia ella hasta que empiece a pasar las pruebas de estado nuevamente.

* **¿Cuál es el propósito de la Load Balancing Rule? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.**

    Una **Load Balancing Rule** de un Load Balancer se usa para definir la manera de distibuir el tráfico entrante a todas las instancias dentro del *Backend Pool*. En Azure existen tres tipos de sesión de persistencia:

   * **None (hash-based):** Especifica que las solicitudes sucesivas del mismo cliente pueden ser manejadas por cualquier máquina virtual. Los paquetes de la misma sesión TCP o UDP se dirigirán a la misma instancia de IP del Datacenter (DIP), pero cuando el cliente cierra y vuelve a abrir la conexión o inicia una nueva sesión desde la misma IP de origen, el puerto de origen cambia y hace que el tráfico vaya a un DIP diferente.
   
        ![](https://media.discordapp.net/attachments/352624122301513730/772882047517065286/MicrosoftTeams-image_7.png)

   * **Client IP (source IP affinity 2-tuple o 3-tuple):** Especifica que las peticiones sucesivas de la misma dirección IP del cliente serán gestionadas por la misma máquina virtual. Azure Load Balancer se puede configurar para usar 2 tuplas (IP de origen, IP de destino) o 3 tuplas (IP de origen, IP de destino, Protocolo) para asignar el tráfico a los servidores disponibles.

        ![](https://media.discordapp.net/attachments/352624122301513730/772882079343181864/MicrosoftTeams-image_6.png)

* **¿Qué es una Virtual Network? ¿Qué es una Subnet? ¿Para qué sirven los address space y address range?**

    * **Virtual Network:** Es una tecnología de red  que permite extender la red de área local sobre una red pública o no controlada (Internet). permite enviar y recibir datos sobre redes compartidas o públicas comportandose como una red privada aprovechando la funcionalidad, seguridad y políticas de gestión de una red privada. se implementan realizando conexiones dedicadas y/o cifrado.
    
    * **Subnet:** Es una segmentación de la red virtual (o cualquier red en general), permiten asignar una o varias subredes a la misma, estas subredes cuentan con un rango de direcciones apropiadas para una organización adecuada.
    
    * **Address space:** Cuando se crea una red virtual, se debe especificar un rango de direcciones ip privadas personalizadas (RFC 1918). Azure asigna a los recursos de una red virtual una dirección IP privada desde el espacio de direcciones que asigne. Por ejemplo, si implementa una máquina virtual en una red virtual con espacio de direcciones, 10.0.0.0/16, a la máquina virtual se le asignará una dirección IP privada como 10.0.0.4.
    
    * **Address range:** Indica cuantas direcciones se tienen en un address space y dependiendo de la cantidad de recursos que se necesiten en la red virtual, el rango aumentará o disminuirá.

* **¿Qué son las Availability Zone y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea zone-redundant?**

    * **Zonas de disponibilidad:** Son Ubicaciones geográficas únicas dentro de una región. Cada zona se compone de uno o más centros de datos equipados con alimentación, refrigeración y redes independientes. Seleccionamos 3 zonas de disponibilidad diferentes para poder tener una mejor disponibiliad y tolerancia a fallos dentro del sistema. En caso de que falle alguno de los centros de datos anteriormente mencionados, el loadbalancer utilizará otro nodo de la red que se encontrará ubicado en otra ubicación geográfica, de esta manera se garantiza resiliencia y se disminuye la probabilidad de que el sistema se encuentre no disponible.
    
    * **IP zone-redundant:** Un gateway zone-redundant aporta resistencia, escalabilidad y disponibilidad a nuestro sistema, cuando utilizamos una ip zone-redundant azure separa física y lógicamente el gateway dentro de una region, lo cual permite mejorar la conectividad de la red privada y disminuye fallos a nivel de zona de disponibilidad.
    
* **¿Cuál es el propósito del Network Security Group?**

    Permite filtrar el tráfico hacia y desde los recursos en una red virtual de Azure, un grupo de seguridad permite definir reglas de entrada y/o salida que permitan o denieguen el tráfico de red entrante o saliente de varios tipos de recursos de Azure.
    
* **Informe de newman 1 (Punto 2)**

    [Informe](#Informe)

* **Presente el Diagrama de Despliegue de la solución.**
    ![Despliegue](https://media.discordapp.net/attachments/352624122301513730/772882775346380800/Deployment_Diagram0_1.png?width=686&height=475)

## Referencias

* !(https://docs.microsoft.com/en-us/azure/load-balancer/skus)
* !(https://azure.microsoft.com/en-us/pricing/calculator/)
* !(https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview)
* !(https://azure.microsoft.com/en-us/blog/azure-load-balancer-new-distribution-mode/)
* !(https://docs.microsoft.com/en-us/azure/virtual-network/network-security-groups overview#:~:text=You%20can%20use%20an%20Azure,several%20types%20of%20Azure%20resources.)




