# LABORATORIO 04: SEMAFORO CRONOMETRADO PEATONAL Y VEHICULAR 

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
- Comprender el modelo de Máquina de Estado Finito (FSM) tipo Moore.
- Diseñar e implementar una FSM en Verilog con separación clara de bloques combinacionales y secuenciales.
- Sintetizar y programar el diseño en la tarjeta DE10.Lite (MAX 10).
- Verificar el comportamiento temporal del semáforo mediante simulación y prueba en hardware.


#### MARCO TEÓRICO:

##### Maquinas de estado finito (FSM):
Una Máquina de Estado Finito es un modelo matemático de cómputo secuencial que consta de:
![Imagen_1](./IMG_05/FSM.png) 

Existen dos modelos principales:

Moore: las salidas dependen solo del estado actual. La salida es más estable (menos glitches).
Mealy: las salidas dependen del estado y de las entradas actuales. Puede requerir menos estados.
En este laboratorio se implementa el modelo Moore, ya que cada luz del semáforo corresponde directamente a un estado.

![Imagen_2](./IMG_05/ESTADOS%20SEMAFORO.png)

#### Implementación en Verilog: 
Estructura de Tres Bloques
La FSM en Verilog se implementa con tres bloques always separados para claridad y buenas prácticas de síntesis:

always @(posedge clk) → Registro de estado (bloque secuencial)
always @(*)           → Lógica de transición (bloque combinacional)
always @(*)           → Lógica de salida (bloque combinacional)

Esta separación evita latch inferidos y facilita la verificación.


### RECURSOS Y MATERIALES: 

- Tarjeta de desarrollo - FPGA DE10-Lite (Max10)
- Software Quartus
- Modulo semaforo
 


### SIMULACIONES IMPLEMENTACION EN VERILOG Y CÓDIGOS: 

A continuacion se mostraran los resultados de la sintetizacion en Verilog de las siguientes simulaciones: 
- Módulo principal FSM 
- Divisor de reloj (50 MHz → 1 Hz)
-  Testbench de simulación (Archivo Test Bench) 


#### CODIGO Y SINTESIS IMPLEMENTADO EN VERILOG: 

![Imagen_3](./IMG_05/VERILOG%20SEMAFORO%20.jpeg)

En esta imagen observamos el espectro de cada estado del semaforo en donde se evidencia el estado lógico (0 o 1) todo esto depende del Timer que se le haya colocado en el codigo verilog que se mostrara a continuación: 

##### CODIGO VERILOG: 
###### ARCHIVO.V: 
module semaforo (
    input wire clk,          // Reloj de la tarjeta
    input wire rst,          // Botón de reinicio
    input wire [1:0] sw,     // Switches (por si los usas luego)

    output reg RV,           // Rojo Vehicular
    output reg AV,           // Amarillo Vehicular
    output reg VV,           // Verde Vehicular
    output reg RP,           // Rojo Peatonal
    output reg VP,           // Verde Peatonal
    output reg alarma        // Alarma sonora
);

// ==========================================
// 1. DIVISOR DE FRECUENCIA
// ==========================================
reg [24:0] counter;
reg clk_1HZ;

always @(posedge clk or posedge rst) begin
    if (rst) begin
        counter <= 0;
        clk_1HZ <= 0;
    end else begin
        if (counter == 25000000 - 1) begin 
            counter <= 0;
            clk_1HZ <= ~clk_1HZ;
        end else begin
            counter <= counter + 1;
        end
    end
end

// ==========================================
// 2. DEFINICIÓN DE ESTADOS Y TEMPORIZADOR
// ==========================================
reg [1:0] state;
reg [1:0] next_state;
reg [3:0] timer;

parameter s_rojo              = 2'b00;
parameter s_amarillo_A_verde = 2'b01;
parameter s_verde             = 2'b10;
parameter s_amarillo_V_rojo  = 2'b11;

// ==========================================
// 3. BLOQUE DE CONTROL DE ESTADOS (SECUENCIAL)
// ==========================================
always @(posedge clk_1HZ or posedge rst) begin
    if (rst) begin
        state <= s_rojo;
        timer <= 0;
    end else begin
        state <= next_state;
        if (state != next_state) begin
            timer <= 0;        // Reinicia el tiempo al cambiar de color
        end else begin
            timer <= timer + 1; // Cuenta los segundos
        end
    end
end

// ==========================================
// 4. BLOQUE DE TRANSICIÓN DE ESTADOS (COMBINACIONAL)
// ==========================================
always @(*) begin
    case (state)
        s_rojo: begin
            if (timer >= 4) next_state = s_amarillo_A_verde;
            else            next_state = s_rojo;
        end

        s_amarillo_A_verde: begin
            if (timer >= 2) next_state = s_verde;
            else            next_state = s_amarillo_A_verde;
        end

        s_verde: begin
            if (timer >= 4) next_state = s_amarillo_V_rojo;
            else            next_state = s_verde;
        end

        s_amarillo_V_rojo: begin
            if (timer >= 2) next_state = s_rojo;
            else            next_state = s_amarillo_V_rojo;
        end

        default: next_state = s_rojo;
    endcase
end 

// ==========================================
// 5. BLOQUE DE SALIDAS (COMBINACIONAL)
// ==========================================
always @(*) begin 
    // Valores por defecto seguros
    RV = 0; AV = 0; VV = 0;
    RP = 1; VP = 0; alarma = 0;

    case (state)
        s_rojo: begin
            RV = 1; // Autos detienen
            RP = 0; 
            VP = 1; // Peatones cruzan
            alarma = 1; // Alarma activa para ciegos
        end

        s_amarillo_A_verde: begin
            AV = 1; // Autos advertencia
            RP = 1; // Peatones esperan
            VP = 0;
        end

        s_verde: begin
            VV = 1; // Autos avanzan
            RP = 1; // Peatones esperan
            VP = 0;
        end

        s_amarillo_V_rojo: begin
            AV = 1; // Autos advertencia
            RP = 1; // Peatones esperan
            VP = 0;
        end

        default: begin
            RV = 1;
            RP = 1;
        end
    endcase  
end 

endmodule

##### ARCHIVO.TB: 



### EVIDENCIAS: 



#### VIDEO DE IMPLEMENTACION FISICA: 

A continuacion se encontrara el video explicativo de la implementacion en fisico del cronometro multiplexado.

[Ver video en YouTube](https://www.youtube.com/watch?v=OD0RrbT2hiY)

### CONCLUSIONES: 
1. Optimización drástica de pines de control: La multiplexación permite controlar múltiples displays de 7 segmentos utilizando un solo bus de datos común (7 u 8 pines para los segmentos) y un pin de activación por cada dígito. Esto reduce significativamente los pines necesarios en el microcontrolador o FPGA en comparación con el control estático individual.
2. Engaño visual mediante persistencia retiniana: El sistema aprovecha el fenómeno de la persistencia de la visión humana (POV). Al encender y apagar cada dígito secuencialmente a una frecuencia alta (generalmente superior a 50 Hz), el ojo humano no percibe el parpadeo y ve todos los números encendidos al mismo tiempo.
3. Reducción del consumo de energía y costos: Al estar encendido solo un display a la vez en un instante dado, el consumo de corriente pico disminuye considerablemente en comparación con tener todos los segmentos encendidos en paralelo. Además, reduce la cantidad de resistencias y drivers necesarios, abaratando el circuito.
4. Importancia del temporizador (Timer) y la precisión: Para un cronómetro, la precisión del tiempo es crítica. El uso de interrupciones por temporizador (Timers) es fundamental para garantizar que el incremento del tiempo (centésimas, segundos, minutos) sea exacto y que la conmutación entre los displays sea fluida y constante, sin afectar el conteo.
5. Complejidad del código frente a simplicidad de hardware: Este diseño demuestra un intercambio clásico en ingeniería: se reduce la complejidad del hardware a cambio de mayor complejidad en el software. El programa debe gestionar constantemente el refresco de las pantallas y la conversión de los datos binarios del cronómetro a código de 7 segmentos (BCD a 7 segmentos) en tiempo real.
