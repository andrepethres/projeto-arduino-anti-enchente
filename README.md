# Alarme de Nível de Água com Arduino

Este projeto utiliza um Arduino para monitorar o nível de água e acionar um alarme. O sistema inclui um sensor de nível de água, um buzzer, uma bomba d'água e um LED para indicar o status do nível de água.

## Descrição do Projeto

O alarme de nível de água é projetado para detectar quando o nível da água atinge um limite predefinido. Quando o nível é detectado, o sistema aciona a bomba d'água, ativa um buzzer e acende um LED vermelho para sinalizar a condição de alarme. O buzzer soa em intervalos regulares para alertar sobre o estado crítico.

## Mapeamento de Hardware

- **SENSOR_PIN (5)**: Entrada digital para o sensor de nível de água.
- **BUZZER_PIN (6)**: Saída digital para o buzzer contínuo.
- **BOMBA_PIN (7)**: Saída digital para a bomba d'água.
- **LED_RED_PIN (8)**: Saída digital para o LED vermelho.

## Esquema de Conexões

1. **Sensor de Nível de Água**: Conecte ao pino digital 5 do Arduino.
2. **Buzzer**: Conecte ao pino digital 6 do Arduino.
3. **Bomba d'Água**: Conecte ao pino digital 7 do Arduino.
4. **LED Vermelho**: Conecte ao pino digital 8 do Arduino.

## Código

```cpp
/*
   Curso de Arduino e AVR 105

   WR Kits Channel

   Alarme de Nível de Água

   Autor: Eng. Wagner Rambo  Data: Fevereiro de 2017

   www.wrkits.com.br | facebook.com/wrkits | youtube.com/user/canalwrkits
*/

// =============================================================================================================
// --- Mapeamento de Hardware ---
#define SENSOR_PIN 5         // Sensor de nível de água
#define BUZZER_PIN 6         // Buzzer contínuo 5V
#define BOMBA_PIN 7          // Bomba d'água
#define LED_RED_PIN 8        // LED vermelho

// =============================================================================================================
// --- Protótipo das funções auxiliares ---
void read_sensor();            // Função para leitura do sensor e acionamento da bomba

// =============================================================================================================
// --- Declaração de variáveis globais ---
volatile short counterT2 = 0; // Contador auxiliar para o Timer2

// =============================================================================================================
// --- Interrupção ---
ISR(TIMER2_OVF_vect)
{
    counterT2++;               // Incrementa contador auxiliar do Timer2

    if (counterT2 >= 100) {    // Chegou a 100 contagens?
        counterT2 = 0;         // Reinicia o contador

        // Inverte estado do buzzer (soa alarme)
        digitalWrite(BUZZER_PIN, !digitalRead(BUZZER_PIN));
    }
}

// =============================================================================================================
// --- Configurações iniciais ---
void setup()
{
    // -- Direção dos I/Os --
    pinMode(BUZZER_PIN, OUTPUT);     // Saída para o buzzer
    pinMode(BOMBA_PIN, OUTPUT);      // Saída para a bomba d'água  
    pinMode(SENSOR_PIN, INPUT_PULLUP); // Entrada para o sensor com pull-up interno 
    pinMode(LED_RED_PIN, OUTPUT);    // Saída para LED vermelho

    // -- Inicialização das saídas --
    digitalWrite(BUZZER_PIN, LOW);   // Desliga buzzer
    digitalWrite(BOMBA_PIN, LOW);    // Desliga bomba
    digitalWrite(LED_RED_PIN, LOW);  // Desliga LED vermelho

    // -- Registradores de configuração do Timer2 --
    TCCR2A = 0x00;                // Timer operando em modo normal
    TCCR2B = (1 << CS22) | (1 << CS21) | (1 << CS20); // Prescaler 1:1024
    TCNT2  = 0x00;                // Inicializa o contador do Timer2
    TIMSK2 = (1 << TOIE2);        // Habilita interrupção do Timer2
}

// =============================================================================================================
// --- Loop Infinito ---
void loop()
{
    read_sensor();
}

// =============================================================================================================
// --- Funções Auxiliares ---
void read_sensor()              // Função para leitura do sensor e acionamento da bomba
{
    if (digitalRead(SENSOR_PIN)) { // Atingiu o nível de água limite?
        digitalWrite(BOMBA_PIN, HIGH);   // Liga bomba
        digitalWrite(LED_RED_PIN, HIGH); // Liga LED
        digitalWrite(BUZZER_PIN, HIGH);  // Liga buzzer
    } else {
        digitalWrite(BUZZER_PIN, LOW);   // Desliga buzzer
        digitalWrite(BOMBA_PIN, LOW);    // Desliga bomba
        digitalWrite(LED_RED_PIN, LOW);  // Desliga LED
    }
}
