# Laboratorio 2: Sumador de 4 Bits 

## Integrantes
* [Wilmer Fernando Puentes Gomez](https://github.com/wilmerfepuentesgo-alt)
* [Jhojan Manuel Castro Mendoza](https://github.com/casjho05)
* [Alberto Rodriguez Medina](https://github.com/22561132!)
  
## Informe

Indice:

1. [Documentación]
2. [Simulaciones]
3. [Evidencias]
4. [Conclusiones]
5. [Referencias]

## DOCUMENTACION: 

### Implementacion del circuito de 1 Bit para Sumador de 4 Bits: 

Para crear un sumador de 4 bits, se utilizan cuatro sumadores de 1 bit conectados en serie. Así, el acarreo de salida de un sumador de 1 bit se convierte en el acarreo de entrada del siguiente sumador. Cada bit de los dos números que se están sumando se procesa de manera paralela.

Para construir un sumador de 4 bits utilizando el sumador de 1 bit como módulo base, se debe instanciar varios módulos del sumador de 1 bit y conectar sus entradas y salidas de manera que manejen el acarreo entre cada bit.

Un sumador de 4 bits suma dos números de 4 bits ([3:0] A y [3:0] B) y produce una suma de 4 bits ([3:0] So) y un acarreo de salida (Co). Para lograr esto, se utilizan 4 sumadores de 1 bit, cada uno manejando una posición de la salida So (0 a 3) y el acarreo hacia la siguiente posición. A continuación se muestra su respectivo bloque funcional: 

![Imagen 1](/tecnicasDigitalesG3E1/lab02/img2/SUM4B%20BLOQUES.png)

La implementación del sumador de 4 bits utilizando instancias del sumador de 1 bit es un ejemplo de diseño estructural en HDL, en donde se utiliza el sumador de 1 bit para construir un sumador de 4 bits de manera modular.

### FUNCIONAMIENTO: 
Cada instancia del sumador de 1 bit toma 1 bits de las entradas A y B, y un acarreo de entrada Ci. Calcula la suma de estos bits y produce una suma de un bit So y un acarreo de salida Co.

El acarreo de salida de un sumador de 1 bit se usa como acarreo de entrada para el siguiente sumador de 1 bit en la cadena.

El sumador de 4 bits produce una salida final So de 4 bits y un acarreo de salida final Co.


## SIMULACIONES: 
En este espacio se mostraran tanto los codigos del sumador de 4 Bits en sus archivos .V y TB.V asi como su simulacion en Verilog y su funcionamiento correcto en la tarjeta FPGA. 

### CODIGOS DE LA SIMULACION: 
#### ARCHIVO.V:

`include "SUMADOR.V"
`timescale 1ps/1ps

module SUMADOR4B (
    input [3:0] A, B,
    input Cin,
    output [3:0] S,
    output Cout
);

wire C0;
wire C1;
wire C2;
// Instanciación de los sumadores de 1 bit

SUMADOR BIT0
 (
    .A(A[0]),
    .B(B[0]),
    .Cin(Cin),
    .S(S[0]),
    .Cout(C0)
);

SUMADOR BIT1
 (
    .A(A[1]),
    .B(B[1]),
    .Cin(C0),
    .S(S[1]),
    .Cout(C1)
);

SUMADOR BIT2
 (
    .A(A[2]),
    .B(B[2]),
    .Cin(C1),
    .S(S[2]),
    .Cout(C2)
);

SUMADOR BIT3
 (
    .A(A[3]),
    .B(B[3]),
    .Cin(C2),
    .S(S[3]),
    .Cout(Cout)
);

endmodule

#### ARCHIVO TB.V

`include "SUMADOR4B.v"

`timescale 1ps/1ps

module SUMADOR4B_TB;

    // Señales de estímulo (entradas)
    reg [3:0] A, B;
    reg Cin;
    
    // Señales de monitoreo (salidas)
    wire [3:0] S;
    wire Cout;

    // Conexión con el diseño
    SUMADOR4B uut (
        .A(A),
        .B(B),
        .Cin(Cin),
        .S(S),
        .Cout(Cout)
    );

    initial begin
        // Configuración para GTKWave

        $dumpfile("SUMADOR4B_TB.vcd");
        $dumpvars(0, SUMADOR4B_TB);

        // Casos de prueba (Tabla de verdad)
        A = 4'b0000; B = 4'b0000; Cin = 0; #10;
        A = 4'b0001; B = 4'b0001; Cin = 0; #10;
        A = 4'b0010; B = 4'b0011; Cin = 1; #10;
        A = 4'b0100; B = 4'b0101; Cin = 0; #10;
        A = 4'b0110; B = 4'b0111; Cin = 1; #10;
        A = 4'b1000; B = 4'b1001; Cin = 0; #10;
        A = 4'b1010; B = 4'b1011; Cin = 1; #10;
        A = 4'b1100; B = 4'b1101; Cin = 0; #10;
        A = 4'b1110; B = 4'b1111; Cin = 1; #10;

        $finish;
    end
wire C0;
wire C1;
wire C2;
// Instanciación de los sumadores de 1 bit

SUMADOR BIT0
 (
    .A(A[0]),
    .B(B[0]),
    .Cin(Cin),
    .S(S[0]),
    .Cout(C0)
);

SUMADOR BIT1
 (
    .A(A[1]),
    .B(B[1]),
    .Cin(C0),
    .S(S[1]),
    .Cout(C1)
);

SUMADOR BIT2
 (
    .A(A[2]),
    .B(B[2]),
    .Cin(C1),
    .S(S[2]),
    .Cout(C2)
);

SUMADOR BIT3
 (
    .A(A[3]),
    .B(B[3]),
    .Cin(C2),
    .S(S[3]),
    .Cout(Cout)
);
endmodule

Como se puede analizar se enlazo el archivo del sumador de 1 Bit para asi poder crear a partir de este nuestro sumador de 4 Bits. 

El codigo mostrasdo a continuacion nos muestra una instanciacion con el "for" para asi reducir las lineas de nuestro anterior codigo: 

##### CODIGO CON FOR: 

`include "sumador_bit.v"

`timescale 1ns/1ps

module SUMADOR4B_TB;

    // Entradas del diseño (reg)
    reg [3:0] A;
    reg [3:0] B;
    reg Cin;

    // Salidas del diseño (wire)
    wire [3:0] S;
    wire Cout;

    integer i,j;

    // Instancia de la unidad bajo prueba (UUT)
    sumador_bit uut (
        .A(A),
        .B(B),SUMADOR4B_TB.vcd
        .Cin(Cin),
        .S(S),
        .Cout(Cout)
    );

    initial begin
        // Crear el archivo de ondas para GTKWave
        $dumpfile("");
        $dumpvars(0, SUMADOR4B_TB);

Cin=0;
for(i=0; i<16; i=i+1) begin
    for(j=0; j<16; j=j+1) begin
        A = i; // Asignar valor a A
        B = j; // Asignar valor a B
        
        #10; // Esperar 10 unidades de tiempo
    end
end
        $display("Simulación terminada correctamente.");
        $finish;
    end

endmodule


#### SIMULACION EN VERILOG: 
A continuacion veremos la simulacion de nuestro circuto con el FOR anidado en nuestro codigo: 

![Imagen_2](/tecnicasDigitalesG3E1/lab02/img2/WhatsApp%20Image%202026-09-18%20at%2018.06.06.jpeg)

Se observan las posibles combinaciones de nuestro codigo binario (256 combinaciones) de las cuales se muestran 15 sacadas mediante nuestro anidado "FOR". y las restantes 250 se muestran en nuestras entradas A y B. 

#### EVIDENCIAS: 

En el siguiente video (enlace de video), se observa el correcto funcionamiento de nuestro circuito SUMADOR DE 4 BITS: 

##### LINK DEL VIDEO: 
![Video](/tecnicasDigitalesG3E1/lab01/img/WhatsApp%20Video%202026-08-30%20at%2016.16.49%20(1).mp4)

Link del video en You Tube: 
[Ver video en YouTube](https://youtube.com/shorts/5yb6FERX9uY)


### CONCLUSIONES: 
- El diseño demostró las ventajas del modelado estructurado en Verilog. Permitirá construir el sumador de 4 bits instanciando cuatro módulos de un bit (Full Adder) en cascada, lo que simplifica la depuración del código y refleja cómo el software de síntesis traduce descripciones textuales en bloques de hardware interconectados.

- El uso de las herramientas de simulación de Quartus (como el Waveform Editor o ModelSim) permitió comprobar visualmente el comportamiento del circuito ante diferentes combinaciones de entrada. Esto demostró la importancia de la fase de verificación para asegurar que el diseño cumple con la tabla de verdad antes de realizar cualquier implementación física.

- A través de la simulación temporal, se puede observar cómo los cambios en los bits más significativos (MSB) dependen de la propagación del acarreo de los bits previos. En Quartus, esto evidencia que el código Verilog genera una cadena de retrasos físicos reales (Gate Delays) que afectan el tiempo de establecimiento del resultado final.

- Verilog permite diseñar el sumador tanto a nivel estructural (conectando compuertas and, or, xor) como a nivel de comportamiento (usando el operador aritmético +). Esto concluye que las herramientas modernas de síntesis de Quartus son capaces de optimizar automáticamente las ecuaciones lógicas basándose en descripciones abstractas de alto nivel.

- Al compilar el proyecto en Quartus, el reporte de síntesis muestra el uso exacto de recursos (como Logic Elements o LUTs). Esto permite concluir que un sumador de 4 bits en Verilog es un circuito altamente eficiente que consume un impacto mínimo dentro de la arquitectura de un dispositivo de lógica programable (en nuestro caso se implemento en una tarjeta Max II).

