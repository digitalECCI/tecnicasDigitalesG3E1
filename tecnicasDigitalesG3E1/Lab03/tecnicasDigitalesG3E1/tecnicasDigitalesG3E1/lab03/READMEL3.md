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
### Objetivo.
El objetivo de este laboratorio es lograr visualizar el resultado de un sumador de 3 bits en los display 7 segmentos de la tarjeta de desarrollo DE10-Lite.

### Fundamento teórico
#### 1. BCD (Binary Coded Decimal):
BCD significa Décimal Codificado en Binario y representa el sistema de numeración digital en el que podemos representar cada número décimal utilizando 4 bits de números binarios.

Como sabemos hay 10 dígitos en el sistema décimal, para representarlos necesitamos 10 combinaciones de 4 bits binarios.

![Imagen_1](/tecnicasDigitalesG3E1/tecnicasDigitalesG3E1/lab03/img3/Captura%20de%20pantalla%202026-09-20%20145604.png)

Ahora bien, también es posible representar de forma binaria los números décimales del 10 al 15 pero empleando su correspondiente representación en el sistema hexadécimal:

![Imagen_2](/tecnicasDigitalesG3E1/tecnicasDigitalesG3E1/lab03/img3/HEXA%20.png)

#### 2. Display 7 Segmentos: 
El display de siete segmentos es un dispositivo electrónico que consta de siete diodos emisores de luz (LED) dispuestos en un patrón definido; encender una combinación particular de éstos permite representar un dígito décimal o hexadécimal Existen dos tipos de display LED de siete segmentos:

Tipo de cátodo común: en este tipo de display, todos los cátodos de los siete LEDs están conectados entre sí a tierra o 
−Vcc (por lo tanto, cátodo común) y el LED muestra dígitos cuando se suministra un nivel alto a los ánodos individuales.

Tipo de ánodo común: en este tipo de display, todos los ánodos de los siete LEDs están conectados a 
+Vcc (por lo tanto, ánodo común) y el LED muestra dígitos cuando se suministra un nivel al bajo a los cátodos individuales.

En las siguientes figuras se muestra cómo se distribuyen los 7 segmentos en el display cuando se tiene una configuración de ánodo común:

![Imagen_3](/tecnicasDigitalesG3E1/tecnicasDigitalesG3E1/lab03/img3/D7S.png)


## SIMULACIONES: 



