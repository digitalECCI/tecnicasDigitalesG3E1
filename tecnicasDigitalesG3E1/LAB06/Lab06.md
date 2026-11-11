# LABORATORIO 06: CAJA FUERTE CON CERROJO Y SERVOMOTOR EN FPGA 

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

## DOCUMENTACION: 

### 1. OBJETIVOS: 
Al finalizar este laboratorio, el estudiante será capaz de:

Diseñar una Máquina de Estado Finito (FSM) tipo Moore controlada por entradas del usuario (no por tiempo).
Integrar lógica combinacional y secuencial en una solución completa: FSM de control, comparación de una combinación, conteo de intentos y actuación física.
Generar una señal PWM para controlar la posición de un servomotor como cerrojo.
Mostrar mensajes y realimentación en los displays de 7 segmentos.
Sintetizar, programar y demostrar el sistema sobre la tarjeta DE10-Lite (MAX 10). (En nuestro caso se hara montaje en una MAX II).

### 2. MARCO TEÓRICO:

#### 2.1 La caja fuerte como FSM:

El sistema es una cerradura electrónica que valida una combinación ingresada por el usuario. Su comportamiento depende de la secuencia de dígitos introducidos y del número de intentos realizados, por lo que se modela naturalmente como una FSM tipo Moore: cada estado define de forma única las salidas (servo, LEDs y display).

A diferencia del semáforo (Lab 05), cuyas transiciones dependían del tiempo, aquí las transiciones dependen de las entradas del usuario (switches + botón). Es el otro gran tipo de FSM.

#### 2.2 Ingreso de la combinación:
La combinación consta de 3 dígitos, cada uno un valor de 4 bits (0–F).
El usuario fija cada dígito en los switches SW3..SW0 y lo confirma con un pulso del botón KEY0.
Tras los 3 dígitos, la FSM compara la secuencia ingresada contra la combinación secreta (almacenada con localparam).
La comparación se hace al final, con los tres dígitos completos, para no revelar cuál dígito fue el incorrecto.
El sistema dispone de 3 intentos en total.

#### 2.3 Diagrama de Estados: 

![Imagen_01](./IMG%2006/MAQUINA%20ESTADO%20CAJA.png)

#### 2.4 Tabla de estados y salidas:

![Imagen_02](./IMG%2006/ESTADOS%20CAJA%20FUERTE%20.png)

(*) En S_BLOQUEO el LED rojo puede parpadear (1 Hz) reutilizando el divisor de frecuencia.

Mensajes sugeridos (todos representables en 7 segmentos; pueden personalizarse): AccESO (acceso concedido), Int.  n (intentos restantes), BLOQUE (bloqueo).

#### 2.5 Control del servomotor con PWM:

El servo se posiciona con una señal PWM de 50 Hz (periodo de 20 ms), donde el ancho del pulso define el ángulo:

Pulso de 1 ms → ≈ 0° (cerrojo cerrado).
Pulso de x ms → ≈ 90° (cerrojo abierto).
Con el reloj de 50 MHz de la tarjeta:

Periodo (20 ms)  ->  N = 20 ms / 20 ns = 1 000 000 cuentas   (contador de 20 bits)
Pulso 1 ms       ->      1 ms  / 20 ns =    50 000 cuentas
Pulso 2 ms       ->      2 ms  / 20 ns =   100 000 cuentas
La salida PWM está en alto mientras contador < ancho, donde ancho vale 50 000 (cerrado) o 100 000 (abierto). Es el mismo principio del divisor/contador ya trabajado.

#### 2.6 Estructura de tres bloques en Verilog:

La FSM se implementa con la plantilla ya conocida, separando lo secuencial de lo combinacional:

always @(posedge clk) → Registro de estado + registros de dígitos/intentos
always @(*)           → Lógica de próximo estado (case)
always @(*)           → Lógica de salida (servo, LEDs, mensaje del display)
El generador PWM del servo y el decodificador de mensajes van en módulos aparte.

### 3. RECURSOS Y MATERIALES: 

Tarjeta de desarrollo FPGA DE10-Lite (MAX 10).(En nuestro caso se utilizara la MAX II).
Software Quartus.
1 servomotor SG90 (micro servo, 180°).
LED verde y LED rojo externos, con sus resistencias (los LED de la placa son solo rojos).
Fuente de 5 V externa para el servo (power bank o fuente USB) — comparte solo GND con la FPGA.
Protoboard, cables y una caja/maqueta (cartón o impresa 3D) para el cerrojo.

### 4.  Procedimiento
#### Paso 1 — Diseño de la FSM: 

Antes de escribir código, elabore en el informe:

El diagrama de burbujas con los 7 estados y sus salidas (notación Moore).
El diagrama de bloques del sistema (FSM + PWM del servo + decodificador de display).
Las condiciones de transición (pulsos de KEY0, resultado de la comparación, intentos restantes).
La definición de su combinación secreta (3 dígitos de 4 bits).

#### Paso 2 — Implementación en Verilog:

Creen los modulo necesarios que permitan describir la maquia de estados de la caja fuerte, integrando los modulos necesarios para el display 7 segmentos, los divisores de frecuencia y los controles PWM.

#### Paso 3 — Simulación: 

Agregue el testbench al proyecto.
Verifique los tres escenarios:
Combinación correcta → el PWM pasa a 2 ms (servo abierto) y se activa LED verde con AccESO.
Combinación incorrecta → LED rojo, se decrementan los intentos y el display muestra los restantes.
Tres fallos → estado BLOQUEO con el mensaje correspondiente.
Capture las formas de onda para el informe (incluya el ancho del pulso PWM en cada caso).

#### Paso 4 — Implementación física: 

Asigne los pines: SW3..SW0, KEY0 (enter), KEY1 (reset), GPIO del servo y de los LEDs, y los HEX.
Programe la tarjeta y monte el servo en la maqueta como cerrojo.
Demuestre en la sesión: apertura con clave correcta, realimentación de intentos y bloqueo.
Suba el proyecto al repositorio con la documentación y las evidencias.

### 5. SIMULACIONES Y CODIGO EN VERILOG: 

#### ARCHIVO .V: 

// =========================================================================
// MÓDULO PRINCIPAL: CAJA FUERTE PARA CPLD MAX II (EPM240T100C5) + 4 DISPLAYS
// =========================================================================
module caja_fuerte (
    input        clk,         // Reloj de 50 MHz (PIN_12 en la mayoría de MAX II)
    input        btn_enter,   // Botón confirmar dígito (Activo en bajo)
    input        btn_reset,   // Botón reset maestro (Activo en bajo)
    input  [3:0] sw_digit,    // 4 Interruptores para el dígito (0-F)
    
    output       led_open,    // LED verde: Acceso concedido
    output       led_error,   // LED rojo: Error / Parpadeo en Bloqueo
    output       pwm_servo,   // Señal PWM para servomotor SG90 (Cable naranja)
    
    // Salidas para el display multiplexado de 4 dígitos (HS420561K - Cátodo Común)
    output [6:0] seg,         // Segmentos {g, f, e, d, c, b, a} (Activos en ALTO)
    output [3:0] dig          // Selectores {D4, D3, D2, D1} (Activos en BAJO)
);

    wire btn_pulse_sync;
    wire [3:0] sw_sync;
    wire [2:0] estado_actual;
    wire [1:0] intentos_restantes;
    wire [3:0] eco_digito;
    wire ctrl_caja_abierta;
    wire led_green_fsm;
    wire led_red_fsm;

    // ---- Divisor a 1 Hz para parpadeo en BLOQUEO ----
    reg [24:0] clk_1hz_div;
    reg clk_1hz;

    always @(posedge clk or negedge btn_reset) begin
        if (!btn_reset) begin
            clk_1hz_div <= 25'd0;
            clk_1hz     <= 1'b0;
        end else begin
            if (clk_1hz_div >= 25'd24_999_999) begin
                clk_1hz_div <= 25'd0;
                clk_1hz     <= ~clk_1hz;
            end else begin
                clk_1hz_div <= clk_1hz_div + 1'b1;
            end
        end
    end

    // ---- Instanciación de Submódulos ----
    entradas_control inst_entradas (
        .clk       (clk),
        .rst_n     (btn_reset),
        .btn_u     (btn_enter),
        .sw_in     (sw_digit),
        .btn_pulse (btn_pulse_sync),
        .sw_sync   (sw_sync)
    );

    fsm_caja_fuerte inst_fsm (
        .clk          (clk),
        .rst_n        (btn_reset),
        .btn_pulse    (btn_pulse_sync),
        .sw_sync      (sw_sync),
        .estado_act   (estado_actual),
        .caja_abierta (ctrl_caja_abierta),
        .led_green    (led_green_fsm),
        .led_red      (led_red_fsm),
        .intentos     (intentos_restantes),
        .digito_eco   (eco_digito)
    );

    control_pwm inst_pwm (
        .clk          (clk),
        .rst_n        (btn_reset),
        .caja_abierta (ctrl_caja_abierta),
        .pwm_servo    (pwm_servo)
    );

    multiplexor_4_disp inst_mux_disp (
        .clk        (clk),
        .rst_n      (btn_reset),
        .estado_act (estado_actual),
        .intentos   (intentos_restantes),
        .digito_eco (eco_digito),
        .seg        (seg),
        .dig        (dig)
    );

    assign led_open  = led_green_fsm;
    assign led_error = (estado_actual == 3'b110) ? clk_1hz : led_red_fsm;

endmodule

// =========================================================================
// SUBMÓDULO 1: ENTRADAS Y ANTIRREBOTE (10 ms a 50 MHz)
// =========================================================================
module entradas_control (
    input clk,
    input rst_n,
    input btn_u,
    input [3:0] sw_in,
    output reg btn_pulse,
    output reg [3:0] sw_sync
);
    reg [18:0] counter;
    wire clk_slow = (counter == 19'd500_000);

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) counter <= 0;
        else if (clk_slow) counter <= 0;
        else counter <= counter + 1;
    end

    reg [2:0] btn_sr;
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) btn_sr <= 3'b111;
        else if (clk_slow) btn_sr <= {btn_sr[1:0], btn_u};
    end

    wire btn_clean = (btn_sr == 3'b000); 
    reg btn_clean_reg;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            btn_clean_reg <= 1'b0;
            btn_pulse     <= 1'b0;
            sw_sync       <= 4'b0000;
        end else begin
            btn_clean_reg <= btn_clean;
            btn_pulse     <= (!btn_clean) && btn_clean_reg; 
            sw_sync       <= sw_in;
        end
    end
endmodule

// =========================================================================
// SUBMÓDULO 2: FSM MOORE (CLAVE SECRETA: A - 5 - 2)
// =========================================================================
module fsm_caja_fuerte (
    input clk,
    input rst_n,
    input btn_pulse,
    input [3:0] sw_sync,
    output reg [2:0] estado_act,
    output reg caja_abierta,
    output reg led_green,
    output reg led_red,
    output reg [1:0] intentos,
    output reg [3:0] digito_eco
);
    localparam [3:0] CLAVE_D1 = 4'hA;
    localparam [3:0] CLAVE_D2 = 4'h5;
    localparam [3:0] CLAVE_D3 = 4'h2;

    localparam S_INICIO  = 3'b000, S_CAP1 = 3'b001, S_CAP2 = 3'b010;
    localparam S_VERIF   = 3'b011, S_ABIERTO = 3'b100, S_ERROR = 3'b101, S_BLOQUEO = 3'b110;

    reg [3:0] reg_d1, reg_d2, reg_d3;
    reg [2:0] estado_prox;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            estado_act <= S_INICIO;
            intentos   <= 2'd3;
            reg_d1     <= 4'd0; reg_d2 <= 4'd0; reg_d3 <= 4'd0;
            digito_eco <= 4'd0;
        end else begin
            estado_act <= estado_prox;
            case (estado_act)
                S_INICIO: if (btn_pulse) begin reg_d1 <= sw_sync; digito_eco <= sw_sync; end
                S_CAP1:   if (btn_pulse) begin reg_d2 <= sw_sync; digito_eco <= sw_sync; end
                S_CAP2:   if (btn_pulse) begin reg_d3 <= sw_sync; digito_eco <= sw_sync; end
                S_ERROR:  if (intentos > 0) intentos <= intentos - 1'b1;
                S_ABIERTO: intentos <= 2'd3;
            endcase
        end
    end

    always @(*) begin
        estado_prox = estado_act;
        case (estado_act)
            S_INICIO:  if (btn_pulse) estado_prox = S_CAP1;
            S_CAP1:    if (btn_pulse) estado_prox = S_CAP2;
            S_CAP2:    if (btn_pulse) estado_prox = S_VERIF;
            S_VERIF:   estado_prox = ((reg_d1 == CLAVE_D1) && (reg_d2 == CLAVE_D2) && (reg_d3 == CLAVE_D3)) ? S_ABIERTO : S_ERROR;
            S_ABIERTO: estado_prox = S_ABIERTO;
            S_ERROR:   estado_prox = (intentos == 2'd0) ? S_BLOQUEO : S_INICIO;
            S_BLOQUEO: estado_prox = S_BLOQUEO;
            default:   estado_prox = S_INICIO;
        endcase
    end

    always @(*) begin
        caja_abierta = (estado_act == S_ABIERTO);
        led_green    = (estado_act == S_ABIERTO);
        led_red      = (estado_act == S_ERROR) || (estado_act == S_BLOQUEO);
    end
endmodule

// =========================================================================
// SUBMÓDULO 3: PWM PARA SERVOMOTOR SG90 (50 Hz con reloj de 50 MHz)
// =========================================================================
module control_pwm (
    input clk,
    input rst_n,
    input caja_abierta,
    output reg pwm_servo
);
    reg [19:0] cuenta_periodo;
    wire [19:0] ancho_pulso = caja_abierta ? 20'd100_000 : 20'd50_000;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            cuenta_periodo <= 20'd0;
            pwm_servo      <= 1'b0;
        end else begin
            cuenta_periodo <= (cuenta_periodo >= 20'd999_999) ? 20'd0 : cuenta_periodo + 1'b1;
            pwm_servo      <= (cuenta_periodo < ancho_pulso);
        end
    end
endmodule

// =========================================================================
// SUBMÓDULO 4: MULTIPLEXOR DE 4 DISPLAYS (CÁTODO COMÚN: HS420561K)
// =========================================================================
module multiplexor_4_disp (
    input        clk,
    input        rst_n,
    input  [2:0] estado_act,
    input  [1:0] intentos,
    input  [3:0] digito_eco,
    output reg [6:0] seg,     // {g, f, e, d, c, b, a} Activo en ALTO
    output reg [3:0] dig      // {D4, D3, D2, D1} Activo en BAJO
);
    // Contador de barrido (~1.5 kHz con reloj de 50 MHz usando bits [15:14])
    reg [15:0] scan_cnt;
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) scan_cnt <= 16'd0;
        else        scan_cnt <= scan_cnt + 1'b1;
    end

    wire [1:0] sel_digito = scan_cnt[15:14];

    // Caracteres en Cátodo Común (1 = encendido, 0 = apagado) -> {g,f,e,d,c,b,a}
    localparam C_RAYA = 7'b1000000; // '-'
    localparam C_O    = 7'b0111111; // 'O'
    localparam C_P    = 7'b1110011; // 'P'
    localparam C_E    = 7'b1111001; // 'E'
    localparam C_N    = 7'b1010100; // 'n'
    localparam C_I    = 7'b0000110; // 'I'
    localparam C_T    = 7'b1111000; // 't'
    localparam C_B    = 7'b1111100; // 'b'
    localparam C_L    = 7'b0111000; // 'L'
    localparam C_Q    = 7'b1100111; // 'q'

    reg [6:0] seg_eco, seg_int;

    // Decodificador hexadecimal (0-F) para el dígito ingresado
    always @(*) begin
        case (digito_eco)
            4'h0: seg_eco = 7'b0111111; 4'h1: seg_eco = 7'b0000110;
            4'h2: seg_eco = 7'b1011011; 4'h3: seg_eco = 7'b1001111;
            4'h4: seg_eco = 7'b1100110; 4'h5: seg_eco = 7'b1101101;
            4'h6: seg_eco = 7'b1111101; 4'h7: seg_eco = 7'b0000111;
            4'h8: seg_eco = 7'b1111111; 4'h9: seg_eco = 7'b1101111;
            4'hA: seg_eco = 7'b1110111; 4'hB: seg_eco = 7'b1111100;
            4'hC: seg_eco = 7'b0111001; 4'hD: seg_eco = 7'b1011110;
            4'hE: seg_eco = 7'b1111001; 4'hF: seg_eco = 7'b1110001;
            default: seg_eco = C_RAYA;
        endcase
    end

    // Decodificador de intentos restantes
    always @(*) begin
        case (intentos)
            2'd2: seg_int = 7'b1011011; // '2'
            2'd1: seg_int = 7'b0000110; // '1'
            default: seg_int = 7'b0111111; // '0'
        endcase
    end

    // Selección de carácter por cada uno de los 4 dígitos
    reg [6:0] d1_val, d2_val, d3_val, d4_val; // d1 = izquierda, d4 = derecha

always @(*) begin
case (estado_act)
3'b000: begin // INICIO: "----"
d1_val = C_RAYA; d2_val = C_RAYA; d3_val = C_RAYA; d4_val = C_RAYA;
end
3'b001, 3'b010, 3'b011: begin // CAPTURA: "---X"
d1_val = C_RAYA; d2_val = C_RAYA; d3_val = C_RAYA; d4_val = seg_eco;
end
3'b100: begin // ABIERTO: "OPEn"
d1_val = C_O; d2_val = C_P; d3_val = C_E; d4_val = C_N;
end
3'b101: begin // ERROR: "Int2" / "Int1"
d1_val = C_I; d2_val = C_N; d3_val = C_T; d4_val = seg_int;
end
3'b110: begin // BLOQUEO: "bLoq"
d1_val = C_B; d2_val = C_L; d3_val = C_O; d4_val = C_Q;
end
default: begin
d1_val = C_RAYA; d2_val = C_RAYA; d3_val = C_RAYA; d4_val = C_RAYA;
end
endcase
end
// Multiplexación temporal (un dígito encendido a la vez en bajo)
always @(*) begin
case (sel_digito)
2'b00: begin dig = 4'b1110; seg = d1_val; end // Enciende D1 (Izquierda)
2'b01: begin dig = 4'b1101; seg = d2_val; end // Enciende D2
2'b10: begin dig = 4'b1011; seg = d3_val; end // Enciende D3
2'b11: begin dig = 4'b0111; seg = d4_val; end // Enciende D4 (Derecha)
endcase
end
endmodule

#### ARCHIVO .TB: 

`timescale 1ns / 1ps

module caja_fuerte_tb;

    reg clk;
    reg btn_enter;
    reg btn_reset;
    reg [3:0] sw_digit;

    wire led_open;
    wire led_error;
    wire pwm_servo;
    wire [6:0] hex0, hex1, hex2, hex3, hex4, hex5;

    // Instancia de la caja fuerte
    caja_fuerte uut (
        .clk       (clk),
        .btn_enter (btn_enter),
        .btn_reset (btn_reset),
        .sw_digit  (sw_digit),
        .led_open  (led_open),
        .led_error (led_error),
        .pwm_servo (pwm_servo),
        .hex0      (hex0), .hex1 (hex1), .hex2 (hex2),
        .hex3      (hex3), .hex4 (hex4), .hex5 (hex5)
    );

    // Reloj de 50 MHz (período de 20 ns)
    always #10 clk = ~clk;

    // Tarea de pulsación instantánea (solo toma 80 ns)
    task pulsar_enter;
        begin
            btn_enter = 1'b0; #40; // Presiona por 2 ciclos
            btn_enter = 1'b1; #40; // Suelta por 2 ciclos
        end
    endtask

    initial begin
        clk = 1'b0;
        btn_reset = 1'b1;
        btn_enter = 1'b1;
        sw_digit  = 4'h0;

        // Reset inicial rápido
        #100; btn_reset = 1'b0; #100; btn_reset = 1'b1; #100;

        // ---- ESCENARIO 1: INTENTO FALLIDO 1 ----
        $display("[TB] Enviando clave incorrecta 1...");
        sw_digit = 4'h1; pulsar_enter();
        sw_digit = 4'h2; pulsar_enter();
        sw_digit = 4'h3; pulsar_enter();
        #200;

        // ---- ESCENARIO 2: CLAVE CORRECTA (A-5-2) ----
        $display("[TB] Enviando clave correcta (A-5-2)...");
        sw_digit = 4'hA; pulsar_enter();
        sw_digit = 4'h5; pulsar_enter();
        sw_digit = 4'h2; pulsar_enter();
        #500; 

        // Reset maestro rápido para cerrar y probar bloqueo
        btn_reset = 1'b0; #100; btn_reset = 1'b1; #100;

        // ---- ESCENARIO 3: BLOQUEO POR 3 FALLOS ----
        $display("[TB] Forzando bloqueo (3 intentos erroneos)...");
        
        // Error 1
        sw_digit = 4'hF; pulsar_enter(); sw_digit = 4'hF; pulsar_enter(); sw_digit = 4'hF; pulsar_enter(); #200;
        // Error 2
        sw_digit = 4'hF; pulsar_enter(); sw_digit = 4'hF; pulsar_enter(); sw_digit = 4'hF; pulsar_enter(); #200;
        // Error 3 -> Bloqueo automático
        sw_digit = 4'hF; pulsar_enter(); sw_digit = 4'hF; pulsar_enter(); sw_digit = 4'hF; pulsar_enter(); 
        
        #1000;
        $display("[TB] Simulación finalizada con éxito.");
        $finish;
    end

    // Volcado de ondas para GTKWave
    initial begin
        $dumpfile("caja_fuerte_tb.vcd");
        $dumpvars(0, caja_fuerte_tb);
    end

endmodule

#### SINTESIS EN VERILOG: 

![Imagen_03](./IMG%2006/VERILOG%20CAJA%20FUERTE%20.jpeg)

Verificacion de estados e ingreso de claves incorrectas, numero de intentos fallidos, bloqueo y reset del sistema FSM y validacion de clave de tres digitos correcta. 


### 6. PREGUNTAS A RESPONDER: 

1. La FSM es tipo Moore porque sus salidas dependen únicamente del estado actual, no de las entradas presentes. Por ejemplo, la señal que activa el servo o indica “acceso concedido” cambia sólo cuando la máquina pasa a un nuevo estado; mientras permanece en ese estado, la salida se mantiene estable.

En cambio, una FSM Mealy generaría salidas a partir del estado actual y de las entradas actuales. Esto permitiría reaccionar un ciclo antes, pero las salidas podrían cambiar si una entrada cambia dentro del mismo ciclo y son más propensas a glitches.

Cómo se vería como Mealy: las condiciones de transición pasarían también a determinar salidas. Por ejemplo, en vez de esperar a entrar en un estado ACEPTADO para activar servo_enable, la FSM podría generar servo_enable = (estado == ULTIMO_DIGITO_CORRECTO) && boton_ok; así la salida respondería inmediatamente a la combinación estado-entrada. En el diagrama, las salidas Mealy se escriben sobre las flechas de transición, mientras que en Moore se escriben dentro de cada estado.

2. ![Imagen_04](./IMG%2006/FF%20CF.png)

La codificación binaria ahorra flip-flops, pero normalmente requiere lógica combinacional de próximo estado y salidas más compleja. One-hot usa más flip-flops, aunque simplifica la lógica de transición y salida.

3. Conteo de pulsos y PWM
Con un reloj de 50 MHz, cada ciclo dura:
![Imagen_05](./FREQ%20CLK%20.png)

Por tanto:
![Imagen_06](./IMG%2006/CLK.png)

Para un servo estándar, el período PWM suele ser de 20 ms:

20ms / 20ns = 1000000 ciclos

El contador debe alcanzar al menos 999 999, por lo que requiere:

![Imagen_07](./IMG%2006/LOG%20.png)

Es decir, el contador del PWM debe ser de 20 bits, ya que 
2^19=524288
2^19=524288 es insuficiente y 
2^20=1048576
2^20=1048576 
Cubre el período completo. Los anchos de 1 ms y 2 ms equivalen a contar 50 000 y 100 000 ciclos, respectivamente.

Comparación al final
Conviene comparar la combinación completa al final —por ejemplo, cuando se han ingresado los tres dígitos— y no dígito por dígito porque:

Evita revelar información sobre qué dígito fue incorrecto antes de completar la clave.

Permite aplicar una política de seguridad uniforme: el sistema rechaza la clave después de ingresar los tres dígitos, sin importar en qué posición estuvo el error.

Simplifica la FSM: basta un estado de “clave ingresada” y una comparación de los tres registros de 4 bits.

Reduce errores de diseño, ya que no se necesitan estados intermedios que validen parcialmente la clave.

La comparación puede hacerse concatenando los tres dígitos:

clave_ingresada <= digito1 & digito2 & digito3;
clave_correcta  <= x"1A7"; -- ejemplo de 12 bits

acceso_ok <= '1' when clave_ingresada = clave_correcta else '0';

Combinaciones y seguridad
Cada dígito tiene 4 bits, por lo que admite 
2^4=16 valores. Con tres dígitos:

16^3= 4096

Por lo tanto, existen 4 096 combinaciones distintas.

La seguridad aumentaría al incrementar el número de dígitos o los bits por dígito. Por ejemplo:

![Imagen_08](./IMG%2006/DISEÑO.png)

Además, sería conveniente limitar los intentos fallidos, añadir una espera tras varios errores y no mostrar qué dígito fue incorrecto. Así se dificultan los ataques de fuerza bruta.

Sincronización y antirrebote
Las entradas del usuario —botones o interruptores— deben sincronizarse y antirrebotarse antes de usarlas en la FSM porque son señales mecánicas y asíncronas respecto al reloj de la FPGA.

Sincronización: evita violaciones de setup/hold y estados metaestables al muestrear una señal externa con el reloj de 50 MHz.

Antirrebote: elimina los múltiples flancos espurios que produce un botón al hacer contacto mecánico; sin esto, una sola pulsación podría interpretarse como varias.

Estabilidad de la FSM: evita transiciones falsas, conteos incorrectos de dígitos o activaciones accidentales del servo.

Una práctica habitual es pasar cada entrada por dos flip-flops y luego implementar un filtro que exija que la señal permanezca estable durante varios milisegundos antes de considerarla válida.

Alimentación del servo
El servo no debe alimentarse desde la FPGA porque puede demandar corrientes muy superiores a las que un pin GPIO o la regulación de la placa puede entregar de forma segura. Un servo pequeño puede consumir cientos de miliamperios al moverse y más de 1 A al estar cargado o bloqueado; algunos servos alcanzan 2 A o más en stall.

Además, los picos de corriente del motor pueden provocar caídas de tensión, reinicios de la FPGA, ruido en las señales lógicas o daño en los reguladores y pines. Por ello, la señal PWM de la FPGA debe usarse sólo como señal de control, mientras que la alimentación del servo proviene de una fuente externa regulada de 5 V o 6 V, con suficiente corriente y tierra común con la FPGA.

En el montaje, conectaría:

PWM/FPGA → cable de señal del servo

Fuente externa 5–6 V → cable rojo del servo

GND de la fuente externa → GND del servo y GND de la FPGA

No conectar el positivo del servo a 3.3 V ni al pin de 5 V de la FPGA.