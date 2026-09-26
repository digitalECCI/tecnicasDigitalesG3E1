# LABORATORIO 03: IMPLEMENTACION DEL 7 SEGMENTOS 

## Integrantes
* [Wilmer Fernando Puentes Gomez](https://github.com/wilmerfepuentesgo-alt)
* [Jhojan Manuel Castro Mendoza](https://github.com/casjho05)
* [Alberto Rodriguez Medina](https://github.com/22561132!)
  
## INFORME

Indice:

1. [Documentación]
2. [Simulaciones]
3. [Evidencias]
4. [Conclusiones]
5. [Referencias]

### 1. DOCUMENTACION:

#### OBJETIVO:
Implementar un cronómetro digital capaz de contar segundos, décimas, centésimas y milésimas de segundo, visualizando el resultado en el display de 7 segmentos integrado en la FPGA.

#### MARCO TEÓRICO:

##### Lógica Secuencial: 

La lógica secuencial es una rama de la electrónica digital que estudia sistemas cuyo comportamiento depende no solo de las entradas actuales, sino también del historial de estados anteriores. A diferencia de la lógica combinacional —donde la salida es función exclusiva de las entradas presentes— en los sistemas secuenciales la salida evoluciona según la secuencia de eventos pasados, lo que les permite recordar información.

![Imagen_1](./img4/Diagrama%20Bloques.png)

### COMPONENTES Y PRINCIPIOS FUNDAMENTALES:

##### Flip-Flops: 

Los flip-flops son los elementos de memoria básicos de la lógica secuencial. Cada uno almacena un bit de información y mantiene su estado hasta recibir una señal de activación proveniente del reloj del sistema. Los tipos más utilizados son:

Tipo D – Captura el valor de la entrada en el flanco activo del reloj.
Tipo T – Conmuta su estado cada vez que la entrada está activa.
Tipo SR – Permite Set y Reset independientes de su salida.
Tipo JK – Generalización del SR que elimina el estado prohibido.

![Imagen_2](./img4/FLIP%20FLOP.png)



