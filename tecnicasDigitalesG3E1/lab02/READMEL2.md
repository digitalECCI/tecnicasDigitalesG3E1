# Laboratorio 1: Introducción a la lógica combinacional

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

![Imagen 1](/tecnicasDigitalesG3E1/lab01/img/SUM4B%20BLOQUES.png)

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
