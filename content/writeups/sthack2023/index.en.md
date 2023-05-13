---
weight: 4
title: "[Sthack 2023] - Can't seem to make you mine"
date: 2023-05-13T06:49:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["sthack", "hardware", "pico", "rp2040", "clang", "cryptography"]
categories: ["Writeups"]

lightgallery: true
---

## Description

Pour cette épreuve nous avons un boitier qui gère le contrôle d'accès physique à un lieu. L'utilisateur entre son code secret et le boitier génère un OTP envoyé par SMS. Lors de la saisi du bon OTP, la porte s'ouvre.

De ce que j'en ai vu, le microcontrôleur est un Raspberry Pi Pico et une antenne pour envoyer les sms. L'énoncé nous donne le code secret d'un utilisateur et mentionne le fait qu'il n'est pas possible de démonter le boitier, donc attaques hardware impossibles :(

![](/lib/images/writeups/2023_sthack/boitier.png)


## Prise d'informations

Au début de ce challenge, on commence par prendre en compte les sources C, la documentation de l'équipement et l'équipement en lui même. En regardant le code source à disposition, il est plutôt aisé de voir d'où vient la génération de l'OTP :

```c
// Generate a random OTP
// Always remember to free the return at the end
char * generate_otp (char* secret_pin)
{
    unsigned int seed = read_onboard_temperature() + atoi(secret_pin);
    srand(seed);

    int otp = OTP_MIN + rand() % (OTP_MAX + 1 - OTP_MIN); // Somehow, this is how you get a random int between OTP_MIN and OTP_MAX included
    char *code = malloc(OTP_MAXLEN * sizeof(char));
    itoa(otp, code, 10);

    return code;
}
```

Ce snippet appelle plusieurs autres fonctions :
* read_onboard_temperature() : Permet de récupérer la température du capteur embarqué dans le RPi Pico (RP2040) ;
* atoi(secret_pin) : Transforme le pin du patron dans l'énoncé en integer ;

```c
// Get a int based on board temperature
uint16_t read_onboard_temperature() {
    adc_set_temp_sensor_enabled(true);
    adc_select_input(4); // Select adc for board temp

    uint16_t raw = adc_read();
    /*
        const float conversion_factor = 3.3f / (1<<12); 
        float result = raw * conversion_factor;
        float temp = 27 - (result -0.706)/0.001721;
        printf("Temp = %f °C\n", temp);
    */
    return raw;
}
```

Le fameux OTP est donc généré par la température retournée par le capteur. Le premier guess était "facile, il suffit d'aller dans la même pièce que le capteur, récupérer la température et recalculer la seed". Pour ça j'ai demandé à "Ajani" (le créateur du challenge) de me prêter un Raspberry Pi Pico. Je voulais avoir accès à la température du même capteur et du même environnement que celui de prod pour les tests (au final, c'était une bonne idée puisque la version C et des compilateurs sont différents entre le Pico et ceux sur des desktop).

La première question était donc : le retour de `adc_read()` est sous quel format ? La température en Celsius ? La représentation de la tension ? etc.

C'est à ce moment que je me dis qu'il va falloir faire du C et que ça fait très très longtemps :D

![](https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExZmE4ODJhNTU1YmFiMjRjZGY2NzIyOGM1YjdjZmNmMDliODNiMTYyNCZlcD12MV9pbnRlcm5hbF9naWZzX2dpZklkJmN0PWc/cXblnKXr2BQOaYnTni/giphy.gif)

## Environnement de développement Raspberry Pi Pico

Avant de commencer à attaquer le challenge de front, il a fallu installer l'environnement de dev du Pico, pour voir comment est interprété la température. L'environnement de développement du Pico s'est bien déroulé. Il faut tout d'abord installé le SDK :

```bash
yay -S pico-sdk # I use Arch btw
```

Ces liens là m'ont permis d'installer ce que je voulais rapidement :
* https://www.gibbard.me/using_the_raspberry_pi_pico_on_ubuntu/
* https://learnembeddedsystems.co.uk/pico-usb-serial-code
* https://learnembeddedsystems.co.uk/using-the-rp2040-on-board-temperature-sensor 

Finalement un premier snippet qui récupère la température ambiante et l'affiche a été réalisé sans trop de soucis :

```c
#include <stdio.h>
#include "pico/stdlib.h"
#include "hardware/adc.h"

// Core 0 Main Code
int main(void){
    stdio_init_all();

    // Configure ADC
    adc_init();
    adc_set_temp_sensor_enabled(true);
    adc_select_input(4);

    // Primary Core 0 Loop
    while(1){
        uint16_t raw = adc_read();
        const float conversion_factor = 3.3f / (1<<12);
        float result = raw * conversion_factor;
        float temp = 27 - (result -0.706)/0.001721;
        printf("Temp = %f C\n", temp);
        sleep_ms(1000);
    }
}
```

La première chose frappante est que le retour de la fonction `adc_read()` est un chiffre un peu sorti du chapeau qui diminue quand la température augmente.

## Fonctionnement du capteur de température

Pour comprendre plus en profondeur le fonctionnement de ce capteur de température, il faut creuser dans la datasheet :
* https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf

C'est donc une entrée analogique un peu particulière du RP2040 qui retourne la température ambiante. C'est pas le capteur le plus précis, mais pour les besoins du quotidien c'est largement suffisant.

![](/lib/images/writeups/2023_sthack/adc_cara.png)

La sortie "4" fait référence à la ligne `adc_select_input(4);` du snippet au dessus :

![](/lib/images/writeups/2023_sthack/adc_number.png)

Enfin, nous en apprenons un peu plus sur ce fameux retour de `adc_read()`, il s'agit en réalité de la tension en mV. Pour la suite, on s'en fout. 

![](/lib/images/writeups/2023_sthack/adc_datasheet.png)

Un autre fait intéressant est la formule : 

```bash
T = 27 - (ADC_voltage - 0.706)/0.001721
```

Qui fait beaucoup penser à la formule commentée dans le code de ce petit device :

```c
const float conversion_factor = 3.3f / (1<<12); 
float result = raw * conversion_factor;
float temp = 27 - (result -0.706)/0.001721;
printf("Temp = %f °C\n", temp);
```

## Hypothèse 1 : Bruteforcer les température

Maintenant qu'il est possible de capter la température ambiante, il est possible d'implémenter la génération d'un OTP. Au final, l'idée c'était pas top puisque le RP2040 est dans un boitier fermé et sans ventilateur. Pifer la bonne température à 5 chiffres de décimal, c'est un poil ambitieux. Il a donc fallu trouver une autre solution.

![](https://media.giphy.com/media/xCM0GuXe7bb7a/giphy.gif)

## Hypothèse 2 : Influencer la température

Quelque chose de possible est de contrôler la température du capteur si nous avons du chaud ou du froid. Si la température est fixe et connue, alors il est possible de prédire cet OTP. Il est plus facile et moins destructeur de générer du froid, que du chaud. 

Donc le plan d'action est le suivant :
1. Implémenter une routine qui donne la température, le "raw" et un code ;
2. Inverser l'opération mathématique pour récupérer un code depuis une température ;
3. Implémenter des codes potentiels sur une plage de degrés ;
4. Bombarder de froid au moment de la génération de l'OTP et avant son envoie ;
5. Tester nos candidats.

La première option est plutôt simple, puisqu'elle découle de l'hypothèse précédente. Par contre, les maths c'était moins fun, mais voici ce qu'il en est ressorti :

> raw = (0.706 + (27 - temp) * 0.001721) / (3.3f / (1<<12));

Par chance, l'équipe orgas avait des bombes de froid allant de -30 à -35 degrés Celsius. Il est donc possible de générer des codes pour ces plages de températures :

> TempSense.c

```c
#define OTP_MIN 100000
#define OTP_MAX 999999
#define OTP_MAXLEN 10

char* generate_code(uint16_t raw, char* secret_pin) {
	unsigned int seed = raw + atoi(secret_pin);
	srand(seed);
	int otp = OTP_MIN + rand() % (OTP_MAX + 1 - OTP_MIN); // Somehow, this is how you get a random int between OTP_MIN and OTP_MAX included
	char *code = malloc(OTP_MAXLEN * sizeof(char));
	itoa(otp, code, 10);
	return code;
}

int main(void){
    stdio_init_all();
    char *secret_pin = "324092";
    // Configure ADC
    adc_init();
    adc_set_temp_sensor_enabled(true);
    adc_select_input(4); // Select adc for board temp

    while(1) {
		uint16_t raw;
		for(float temp = -35; temp<-30; temp+=0.01) {
			raw = (0.706 + (27 - temp) * 0.001721) /(3.3f / (1<<12));
			char * code = generate_code(raw, secret_pin);
			printf("temp = %f | ", temp);
			printf("raw = %d | ", raw);
			printf("code : %s\n", code);
        }
		sleep_ms(1000);
    }
}
```

> CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.12)

include(pico_sdk_import.cmake)

project(pico-TempSense)

pico_sdk_init()

add_executable(TempSense
    TempSense.c
)

target_link_libraries(TempSense
    pico_stdlib
    hardware_adc)

pico_enable_stdio_usb(TempSense 1)
pico_enable_stdio_uart(TempSense 0)

pico_add_extra_outputs(TempSense)
```

```bash
cmake . && make -j8
sudo mount /dev/sdar1 ../a
sudo cp TempSense.uf2 ../a
```

Après un reboot, le port série est accessible :

```bash
sudo python ./miniterm.py /dev/ttyACM0 115200
--- Miniterm on /dev/ttyACM0  115200,8,N,1 ---
--- Quit: Ctrl+] | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---
temp = -35.000000 | raw = 1008 | code : 579843
temp = -34.990002 | raw = 1008 | code : 579843
[...]
```

Pour des raisons de lisibilité, il n'y a pas tous les résultats, mais ils sont dispos à la fin du writeup. Ce qu'on peut remarquer, c'est qu'un code est le même pour une variation d'environ 0.5 °C, donc peut de codes à tester sur 5 degrés (de -30 à -35).
On stock tout ça dans un fichier et après un peu de bash-fu on obtient une liste de candidat :

```bash
cat codes.txt | cut -d ' ' -f 11 | sort | uniq 

206046
236550
267054
297558
488331
518835
549339
579843
854265
884769
915273
```

Le code OTP est généré avant l'affichage de "sending OTP...", donc avant de valider le PIN secret, il faut refroidir la puce avec une bombe de froid. 

![](/lib/images/writeups/2023_sthack/froid.png)

Après l'avoir aspergée pendant une dizaine de seconde, le code suivant a fonctionné : **549339**
Ce code correspond à une température entre **-33.710217** et **-33.260292**.

## Flag

![](/lib/images/writeups/2023_sthack/flag.png)

> S4N_FR4NC1SC0

---

## Manuel OTP

```md
This guide describe how the 2FA module works and how to use it.


## User : How to use the 2FA module
When working under normal condition, the 2FA module green LED should be turn on and fixed. This mean the module is waiting for user interraction to start an authentication sequence, click on button any touch of the keypad to start interacting with it.

The module green LED should no be blinking, you can now enter your secret PIN (SPIN), and validate by pressing the touch `#`. 
If you entered a valid secret PIN, the green LED will be on of 1 seconds, then the red and green LEDs will both be rapidly blinking while the module generate and transmit the One Time Password (OTP).
If you entered an invalid secret PIN, the red LED will go on for 5 seconds, then you can try again. After 3 failed attempt, the authentication session will fail, and the module will be blocked for 30 seconds with the red LED on, before going back to his initial state.

Once the OTP have been transmitted, you have 3 minutes to enter it. During this period, the red LED will be blinking indicating the module is waiting for you to enter the OTP. Same as for the SPIN, enter the OTP then validate by pressing `#`.

If you provided an invalid OTP, the red LED will stay on for 3 seconds, you cannot provide another OTP during this period. You have 10 attempts, then the module will considère this OTP as invalid.

If you failed to provide any valid OTP after 3 minutes, or if you have exhausted your 10 attempts, the red light will go on for 30 seconds, then the module will end the authentication sequence and go back to his original state, waiting for a new user interraction to start a new authentication sequence.

If you provided the correct OTP, the signal lines will go on and and both green and red lights will go on for 15 seconds, indicating a valid authentication sequence. At the end of those 15 seconds the module will go back to his original state and wait for a new authentication sequence to begin.


## LED State and signification
This document describe the different LEDs states on the 2FA module and their meaning.

| Red LED state | Green LED state | Signification |
| --- | --- | --- |
| Off | On, fixed | 2FA module is idle and wait for an interraction to start a new authentication sequence |
| Off | On, blinking every 250ms | 2FA module is waiting for user SPIN |
| On, fixed for 5 seconds | Off | Invalid SPIN |
| Off | On, fixed for 3 seconds | Valid SPIN |
| On, blinking every 100ms | On, blinking every 100ms | 2FA module is busy working and transmitting OTP through SMS and cannot interract with user |
| On, blinking every 250 ms | Off | 2FA module is waiting for the OTP that have been transmitted through SMS |
| On, fixed for 3 seconds | Off | Invalid OTP |
| On, fixed for 30 seconds | Off | Failed to provide a valid OTP or SPIN |
| On, fixed for 15 seconds | On, fixed for 15 seconds | Valid OTP and SPIN, access granted, the door is unlock |


## Electronical drawings
You should have had them, but my computer litterally died with all the kicad drawings inside...
```

## Sources du challenge

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include "pico/stdlib.h"
#include "hardware/gpio.h"
#include "hardware/structs/sio.h"
#include "hardware/uart.h"
#include "hardware/irq.h"
#include "hardware/adc.h"
#include "./libs/pico-keypad4x4/pico_keypad4x4.h"
#include "./libs/pico-lcd1602/pico_lcd1602.h"

#define FLAG "REPLACE_WITH_FLAG"

// We must impose a 5 seconds timeout on boot to prevent bruteforce by turning power off/on
#define BOOT_TIMEOUT 5 * 1000

// Trigger lines to be turn on when an authentication session succeed
#define LINE_5V_LOW 16
#define LINE_5V_HIGH 17
#define LINE_12V_LOW 18
#define LINE_12V_HIGH 19
#define LINE_24V_LOW 20
#define LINE_24V_HIGH 21

// Leds
#define NO_LED -1
#define LED_GREEN 2
#define LED_RED 3

// Serial info
#define UART_ID uart0
#define BAUD_RATE 9600
#define UART_TX_PIN 0
#define UART_RX_PIN 1
#define UART_DATA_BITS 8
#define UART_STOP_BITS 1
#define UART_PARITY UART_PARITY_NONE

#define AT_PARSER_START 0
#define AT_PARSER_WAIT_RESP 1
#define AT_PARSER_ERROR 2
#define AT_PARSER_SUCCESS 3

#define UART_READ_BUFFER_SIZE 1000


// LCD screen conf
#define LCD_I2C_CHANNEL i2c1
#define LCD_I2C_SDA 26
#define LCD_I2C_SCL 27


// Constants for authentication delay
#define AUTH_SUCCESS_DELAY 30 * 1000
#define AUTH_FAILED_DELAY 30 * 1000

// Constants for SPIN reading 
#define SPIN_MAXLEN 10
#define STATE_SPIN_WAITING 0
#define STATE_SPIN_SUCCESS 1
#define STATE_SPIN_FAILED 2
#define SPIN_MAX_TRY 3
#define SPIN_TIME_WINDOW 3 * 60 * 1000 * 1000 // 1 minute in microseconds
#define INVALID_SPIN_DELAY 5 * 1000
#define VALID_SPIN_DELAY 1 * 1000

// Constants for OTP reading 
#define OTP_MAXLEN 10
#define STATE_OTP_WAITING 0
#define STATE_OTP_SUCCESS 1
#define STATE_OTP_FAILED 2
#define OTP_MAX_TRY 10
#define OTP_TIME_WINDOW 3 * 60 * 1000 * 1000 // 3 minutes in microseconds
#define INVALID_OTP_DELAY 3 * 1000

// OTP range
#define OTP_MIN 100000
#define OTP_MAX 999999

// Struct and list of all couple of secret pin + phone numbers
#define MAX_USERS 100
typedef struct user user;
struct user
{
    char secret_pin[SPIN_MAXLEN + 1]; // Int from 100 000 to 999 999
    char phone_number[15]; // Phone number format E.164 +XXXXXXXX...
};

const user * users[MAX_USERS] = {
    &((user) {"324092", "+33612345678"}),
};


// A struct to store a pattern for led blinking
// set led to -1 to not blink, give GPIO pin number to blink
// led will bo on for duration ms, off for duration ms
typedef struct blinking_pattern blinking_pattern;
struct blinking_pattern
{
    int led_green;
    int led_red;
    int duration;
};

// Define all led blinking patterns
const blinking_pattern BLINKING_WAITING_USER_SPIN = {LED_GREEN, NO_LED, 250};
const blinking_pattern BLINKING_WAITING_USER_OTP = {NO_LED, LED_RED, 250};
const blinking_pattern BLINKING_TRANSMIT_OTP = {LED_GREEN, LED_RED, 100};

// A variable to store the current blink pattern to use
volatile blinking_pattern blink_pattern;

// Keypad buttons
uint KEYPAD_COLUMNS[4] = {10, 11, 12, 13};
uint KEYPAD_ROWS[4] = {6, 7, 8, 9};
char KEYPAD_MATRIX[16] = {
    '1', '2' , '3', 'A',
    '4', '5' , '6', 'B',
    '7', '8' , '9', 'C',
    'x', '0' , '#', 'D'
};

// A counter to know how many auth session we have done
static int auth_session_counter = 0;

// Variables for uart communication
volatile int at_parser_state = AT_PARSER_START;
static char uart_read_buffer[UART_READ_BUFFER_SIZE] = "";

// RX interrupt handler
void on_uart_rx() {
    while (uart_is_readable(UART_ID)) {
        uint8_t ch = uart_getc(UART_ID);
        char chstr[2] = {ch, '\0'};
        strncat(uart_read_buffer, chstr, UART_READ_BUFFER_SIZE - 1);

        printf("INPUT RX: ASCII %d -> %c\n", ch, ch);
    }

    char * line_end = strstr(uart_read_buffer, "\r\n");
    while (line_end != NULL)
    {
        char line[UART_READ_BUFFER_SIZE] = "";
        strncpy(line, uart_read_buffer, line_end - uart_read_buffer);
        printf("Line: %s\n", line);

        char * command_token = "AT";
        if (strncmp(line, command_token, strlen(command_token)) == 0) // If string start with command token
        {
            at_parser_state = AT_PARSER_WAIT_RESP;
            printf("Find command %s\n", line);
        }
        else if (at_parser_state == AT_PARSER_WAIT_RESP)
        {
            if (strstr(line, "ERROR") != NULL)
            {
                printf("Command failed\n");
                at_parser_state = AT_PARSER_ERROR;
            }
            else if (strstr(line, "OK") != NULL)
            {
                printf("Command success\n");
                at_parser_state = AT_PARSER_SUCCESS;
            }
            else
            {
                printf("Ignore line %s\n", line);
            }
        }
        else
        {
            printf("Ignore line %s\n", line);
        }

        strncpy(uart_read_buffer, line_end + 2, UART_READ_BUFFER_SIZE - 1); // Remove line from buffer
        line_end = strstr(uart_read_buffer, "\r\n");
        printf("New state: %d\n", at_parser_state);      
    }
}


// Init all gpios used by the program
void init_gpios()
{
    pico_keypad_init(KEYPAD_COLUMNS, KEYPAD_ROWS, KEYPAD_MATRIX);

    adc_init();

    // Turn board led on
    gpio_init(PICO_DEFAULT_LED_PIN);
    gpio_set_dir(PICO_DEFAULT_LED_PIN, GPIO_OUT);
    gpio_put(PICO_DEFAULT_LED_PIN, 1); 

    gpio_init(LED_GREEN);
    gpio_set_dir(LED_GREEN, GPIO_OUT);
    gpio_pull_down(LED_GREEN);
    
    gpio_init(LED_RED);
    gpio_set_dir(LED_RED, GPIO_OUT);
    gpio_pull_down(LED_RED);

    gpio_init(LINE_5V_LOW);
    gpio_set_dir(LINE_5V_LOW, GPIO_OUT);
    gpio_pull_up(LINE_5V_LOW);
    gpio_put(LINE_5V_LOW, 1);

    gpio_init(LINE_12V_LOW);
    gpio_set_dir(LINE_12V_LOW, GPIO_OUT);
    gpio_pull_up(LINE_12V_LOW);
    gpio_put(LINE_12V_LOW, 1);

    gpio_init(LINE_24V_LOW);
    gpio_set_dir(LINE_24V_LOW, GPIO_OUT);
    gpio_pull_up(LINE_24V_LOW);
    gpio_put(LINE_24V_LOW, 1);

    gpio_init(LINE_5V_HIGH);
    gpio_set_dir(LINE_5V_HIGH, GPIO_OUT);
    gpio_pull_down(LINE_5V_HIGH);

    gpio_init(LINE_12V_HIGH);
    gpio_set_dir(LINE_12V_HIGH, GPIO_OUT);
    gpio_pull_down(LINE_12V_HIGH);

    gpio_init(LINE_24V_HIGH);
    gpio_set_dir(LINE_24V_HIGH, GPIO_OUT);
    gpio_pull_down(LINE_24V_HIGH);

    // UART to communicate with GSM module
    uart_init(UART_ID, BAUD_RATE);
    uart_set_translate_crlf(UART_ID, false);
    gpio_set_function(UART_TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(UART_RX_PIN, GPIO_FUNC_UART);
    uart_set_hw_flow(UART_ID, false, false); // Disable hardware flow control
    uart_set_format(UART_ID, UART_DATA_BITS, UART_STOP_BITS, UART_PARITY); // Define UART format so we know how to read data
    uart_set_fifo_enabled(UART_ID, false); // Disable FIFO because we will send and receive char by char

    // We will setup an interrupt handler to read RX data
    int UART_IRQ = UART_ID == uart0 ? UART0_IRQ : UART1_IRQ;
    irq_set_exclusive_handler(UART_IRQ, on_uart_rx);
    irq_set_enabled(UART_IRQ, true);

    // We enable interrupt on UART so it trigger the interrupt on RX data
    uart_set_irq_enables(UART_ID, true, false);

    // LCD
    lcd1602_init(LCD_I2C_CHANNEL, LCD_I2C_SDA, LCD_I2C_SCL);
    lcd_init();
}


// Read the code typed by the user
// Always remember to free return at the end
char* user_input(int len, int timeout_us)
{
    absolute_time_t start = get_absolute_time();
    char *code = malloc(len * sizeof(char));
    code[0] = '\0';

    while (1)
    {
        // If we reached timeout, end now
        if (timeout_us != -1 && absolute_time_diff_us(start, get_absolute_time()) > timeout_us)
        {
            break;
        }

        char ch = pico_keypad_get_key();
        if (ch == 0)
        {
            continue;
        }

        if (ch == '#')
        {
            break;
        }

        char chstr[2] = {ch, '\0'};
        lcd_string(chstr);

        strncat(code, chstr, len - 1);

        // If user input max length reached, end there
        if (strlen(code) == len)
        {
            break;
        }
        
        // Wait to do a simple debounce on the buttons
        sleep_ms(250);
        continue;
    }

    return code;
}


// Wait until user press any key on keypad
void wait_for_user_interraction()
{
    while (1)
    {
        char ch = pico_keypad_get_key();
        if (pico_keypad_get_key() != 0)
        {
            break;
        }

        sleep_ms(10);
    }
}


/* References for this implementation:
 * see rp2040-datasheet.pdf 4.9.5. Temperature Sensor
 * pico-examples/adc/adc_console/adc_console.c */



// Get a int based on board temperature
uint16_t read_onboard_temperature() {
    adc_set_temp_sensor_enabled(true);
    adc_select_input(4); // Select adc for board temp

    uint16_t raw = adc_read();
    /*
        const float conversion_factor = 3.3f / (1<<12); 
        float result = raw * conversion_factor;
        float temp = 27 - (result -0.706)/0.001721;
        printf("Temp = %f °C\n", temp);
    */

    return raw;
}


// Generate a random OTP
// Always remember to free the return at the end
char * generate_otp (char* secret_pin)
{
    unsigned int seed = read_onboard_temperature() + atoi(secret_pin);
    srand(seed);

    int otp = OTP_MIN + rand() % (OTP_MAX + 1 - OTP_MIN); // Somehow, this is how you get a random int between OTP_MIN and OTP_MAX included
    char *code = malloc(OTP_MAXLEN * sizeof(char));
    itoa(otp, code, 10);

    return code;
}


// Transmit OTP through SMS
bool transmit_otp(char * otp, char * destination)
{
    printf("Try send OTP to phone %s", destination);

    // Make sure the module is up and running
    uart_puts(UART_ID, "AT\r\n");
    sleep_ms(10);
    uart_puts(UART_ID, "AT\r\n");
    sleep_ms(10);
    uart_puts(UART_ID, "AT\r\n");
    sleep_ms(10);
    printf("UART 1");

    // Check Pin
    uart_puts(UART_ID, "AT+CPIN?\r\n");
    sleep_ms(100);

    // Ensure modem is in text mode
    uart_puts(UART_ID, "AT+CMGF=1\r\n");
    sleep_ms(10);

    // Disable verbose error
    uart_puts(UART_ID, "AT+CMEE=0\r\n");
    sleep_ms(100);
   
    // Forge the SMS command
    char * command_string = "AT+CMGS=\"%s\"\r";
    int command_size = snprintf(NULL, 0, command_string, destination) + 1;
    char command[command_size];
    snprintf(command, command_size, command_string, destination);

    char * sms_string = "Votre code OTP : %s. Ne le transmettez a personne.%c";
    int sms_size = snprintf(NULL, 0, sms_string, otp, 26) + 1;
    char sms[sms_size];
    snprintf(sms, sms_size, sms_string, otp, 26); //%c -> 26 = CTRL + Z char to indicate the end of message to sim800L

    // Send the SMS
    uart_puts(UART_ID, command);
    sleep_ms(10);
    uart_puts(UART_ID, sms);
    sleep_ms(10);

    // Wait for module response to be completed
    while (at_parser_state == AT_PARSER_WAIT_RESP)
    {
        tight_loop_contents();
    }

    return (at_parser_state == AT_PARSER_ERROR ? false : true);
}


// A timer to blink leds depending on the state of the module
bool repeating_led_blinking_timer_callback (struct repeating_timer *timer)
{
    if (blink_pattern.led_green != -1)
    {
        // sio_hw->gpio_togl expect a mask of bits where 1 indicate the gpio must be toogle and 0 no change
        // for example the mask b...0010 will togl GPIO 1, the mask b...0100 will togl GPIO 3, b...0110 togl GPIO 1 et GPIO 3
        // We can calculate our mask by shifting left 1 by the int number of the GPIO Pin
        sio_hw->gpio_togl = (uint32_t) (1 << blink_pattern.led_green);
    }

    if (blink_pattern.led_red != -1)
    {
        sio_hw->gpio_togl = (uint32_t) (1 << blink_pattern.led_red);
    }

    return true;
}


// Check if a secret pin match one of users, if true return user index
user * find_user_for_spin(char * secret_pin)
{
    int i;
    for (i = 0; i < MAX_USERS; i++)
    {
        // Ignore null pointers
        if (users[i] == NULL)
        {
            continue;
        }

        if (strncmp(secret_pin, users[i]->secret_pin, SPIN_MAXLEN) == 0)
        {
            // Do casting to prevent warning on const
            return (user *) users[i];
        }
    }

    return NULL;
}


// Function to be called when an authentication have been successfull
void on_successfull_authentication ()
{
    printf("Auth session %d succeed.\n", auth_session_counter);
    lcd_clear();
    lcd_set_cursor(0, 0);
    lcd_string("Success ! Flag :");
    lcd_set_cursor(1, 0);
    lcd_string(FLAG);

    gpio_put(LED_GREEN, 1);
    gpio_put(LED_RED, 1);

    // Turn trigger lines on
    gpio_put(LINE_5V_LOW, 0);
    gpio_put(LINE_5V_HIGH, 1);
    gpio_put(LINE_12V_LOW, 0);
    gpio_put(LINE_12V_HIGH, 1);
    gpio_put(LINE_24V_LOW, 0);
    gpio_put(LINE_24V_HIGH, 1);

    sleep_ms(AUTH_SUCCESS_DELAY);

    // Turn it back off
    gpio_put(LINE_5V_LOW, 1);
    gpio_put(LINE_5V_HIGH, 0);
    gpio_put(LINE_12V_LOW, 1);
    gpio_put(LINE_12V_HIGH, 0);
    gpio_put(LINE_24V_LOW, 1);
    gpio_put(LINE_24V_HIGH, 0);

    gpio_put(LED_GREEN, 0);
    gpio_put(LED_RED, 0);
}


// Function to be called when an authentication have failed
void on_failed_authentication ()
{
    printf("Auth session %d failed.\n", auth_session_counter);
    lcd_clear();
    lcd_set_cursor(0, 0);
    lcd_string("Authentication");
    lcd_set_cursor(1, 0);
    lcd_string("Failed !");

    gpio_put(LED_GREEN, 0);
    gpio_put(LED_RED, 1);
    sleep_ms(AUTH_FAILED_DELAY);

    gpio_put(LED_RED, 0);
}


int main()
{
    stdio_init_all();

    // Sleep on boot to prevent power off bruteforce attack
    sleep_ms(BOOT_TIMEOUT);

    // Init GPIOs
    init_gpios();

    // Init the repeating timer we will use to managed led blinking
    struct repeating_timer blinking_timer;

    // Init the led struct we will use to transmit info to timer about wich led to blink
    blinking_pattern leds_blink;

    while (1)
    {
        auth_session_counter ++;

        // We wait until the user start interracting
        gpio_put(LED_GREEN, 1);
        printf("Waiting for user interraction to start an authentication session...\n");

        lcd_clear();
        lcd_set_cursor(0, 0);
        lcd_string("Waiting...");
        
        wait_for_user_interraction();
        sleep_ms(250); // debounce

        printf("Start authentication session #%d\n", auth_session_counter);
        lcd_clear();


        /* Start reading user SPIN */
        char *secret_pin = NULL;
        user * matching_user;
        int state_reading_spin = STATE_SPIN_WAITING;
        int reading_spin_try = 0;
        absolute_time_t start = get_absolute_time();
        while (state_reading_spin == STATE_SPIN_WAITING)
        {
            // If time window for reading spin have expired
            if (absolute_time_diff_us(start, get_absolute_time()) > SPIN_TIME_WINDOW)
            {
                state_reading_spin = STATE_SPIN_FAILED;
                continue;
            }

            reading_spin_try ++;

            // We start a timer to blink led accordingly
            // We use a negative value for duration, because we want timer to repeat every x ms since pointer start, not after pointer end
            blink_pattern = BLINKING_WAITING_USER_SPIN;
            add_repeating_timer_ms(-BLINKING_WAITING_USER_SPIN.duration, repeating_led_blinking_timer_callback, NULL, &blinking_timer);

            //Read user secret_pin
            printf("Type SPIN: ");

            lcd_set_cursor(0, 0);
            lcd_string("Type secret PIN: ");
            lcd_set_cursor(1, 0); // Go next line to show pin while typed

            int read_timeout = SPIN_TIME_WINDOW - absolute_time_diff_us(start, get_absolute_time()); // Timeout is timewindow - already spent time
            secret_pin = user_input(SPIN_MAXLEN, read_timeout);
            printf("\n");

            // Stop blinking and make sure all led are off
            cancel_repeating_timer(&blinking_timer);
            gpio_put(LED_GREEN, 0);
            gpio_put(LED_RED, 0);

            // If invalid spin
            matching_user = find_user_for_spin(secret_pin);
            if (matching_user == NULL)
            {
                printf("Invalid SPIN: %s\n", secret_pin);
                
                lcd_clear();
                lcd_set_cursor(0, 0);
                lcd_string("Invalid SPIN !");

                gpio_put(LED_RED, 1);
                sleep_ms(INVALID_SPIN_DELAY);
                gpio_put(LED_RED, 0);

                if (reading_spin_try >= SPIN_MAX_TRY)
                {
                    state_reading_spin = STATE_SPIN_FAILED;
                }

                continue;
            }

            // If valid spin, green led and go for transmission of datas
            state_reading_spin = STATE_SPIN_SUCCESS;
        }

        // If we failed to provide a valid SPIN, timeout + red led and go back to wait for a new auth session
        if (state_reading_spin == STATE_SPIN_FAILED)
        {
            // Free secret_pin from memory we wont need it anymore
            free(secret_pin);

            on_failed_authentication();
            continue;
        }

        // If valid spin, turn green led on for a few seconds
        gpio_put(LED_GREEN, 1);
        sleep_ms(VALID_SPIN_DELAY);
        gpio_put(LED_GREEN, 0);


        /* We have a valid PIN, we will now generate and transmit OTP code */
        printf("Start generation and transmission of OTP for SPIN %s\n", secret_pin);
        lcd_clear();
        lcd_set_cursor(0, 0);
        lcd_string("Sending OTP...");

        // We start a timer to blink led accordingly
        blink_pattern = BLINKING_TRANSMIT_OTP;
        add_repeating_timer_ms(-BLINKING_TRANSMIT_OTP.duration, repeating_led_blinking_timer_callback, NULL, &blinking_timer);

        char * one_time_password = generate_otp(secret_pin);
        printf("OTP have been generated.\n");

        // We can now free secret pin we dont need it anymore
        free(secret_pin);

        // Transmit OTP through 
        int success = transmit_otp(one_time_password, matching_user->phone_number);

        // Only if we failed to transmit the password, we immediatly end the auth session and restart everything
        if (success == false)
        {
            printf("OTP transmission failed.\n");

            lcd_clear();
            lcd_set_cursor(0, 0);
            lcd_string("Cannot send SMS, call the admin");
            sleep_ms(5000);

            cancel_repeating_timer(&blinking_timer);
            free(one_time_password);

            on_failed_authentication();
            continue;
        }

        // Stop blinking
        cancel_repeating_timer(&blinking_timer);
        gpio_put(LED_GREEN, 0);
        gpio_put(LED_RED, 0);

        /* Start reading OTP from user */
        char *user_one_time_password;
        int state_reading_otp = STATE_OTP_WAITING;
        int reading_otp_try = 0;
        start = get_absolute_time();
        while (state_reading_otp == STATE_SPIN_WAITING)
        {
            // If time window for reading otp have expired
            if (absolute_time_diff_us(start, get_absolute_time()) > OTP_TIME_WINDOW)
            {
                state_reading_otp = STATE_OTP_FAILED;
                continue;
            }

            reading_otp_try ++;

            // We start a timer to blink led accordingly
            // We use a negative value for duration, because we want timer to repeat every x ms since pointer start, not after pointer end
            blink_pattern = BLINKING_WAITING_USER_OTP;
            add_repeating_timer_ms(-BLINKING_WAITING_USER_SPIN.duration, repeating_led_blinking_timer_callback, NULL, &blinking_timer);

            // Read user OTP
            printf("Type the secret OTP you have received: ");

            lcd_clear();
            lcd_set_cursor(0, 0);
            lcd_string("Type OTP: ");
            lcd_set_cursor(1, 0);

            int read_timeout = OTP_TIME_WINDOW - absolute_time_diff_us(start, get_absolute_time()); // Timeout is timewindow - already spent time
            char * user_one_time_password = user_input(OTP_MAXLEN, read_timeout);
            printf("\n");

            // Stop blinking and make sure all led are off
            cancel_repeating_timer(&blinking_timer);
            gpio_put(LED_GREEN, 0);
            gpio_put(LED_RED, 0);

            // If invalid otp
            if (strncmp(user_one_time_password, one_time_password, OTP_MAXLEN) != 0)
            {
                printf("Invalid OTP: %s\n", user_one_time_password);

                lcd_clear();
                lcd_set_cursor(0, 0);
                lcd_string("Invalid OTP !");

                gpio_put(LED_RED, 1);
                sleep_ms(INVALID_OTP_DELAY);
                gpio_put(LED_RED, 0);

                if (reading_otp_try >= OTP_MAX_TRY)
                {
                    state_reading_otp = STATE_OTP_FAILED;
                }
                else
                {
                    free(user_one_time_password);
                }

                continue;
            }

            // Valid otp
            state_reading_otp = STATE_OTP_SUCCESS;
        }

        // We can free both generated and user provided OTP, we will not use it anymore
        free(user_one_time_password);
        free(one_time_password);

        // If we failed to provide a valid OTP, timeout + red led and go back to wait for a new auth session
        if (state_reading_otp == STATE_OTP_FAILED)
        {
            printf("Failed to provide a correct OTP %d times.\n", OTP_MAX_TRY);

            on_failed_authentication();
            continue;
        }

        // If valid otp, we finally call the on_successfull_authentication callback 
        on_successfull_authentication();
    }

    return 0;
}
```

## Exploit complet

Le code final complet :

```c
#include <stdio.h>
#include <stdlib.h>
#include "pico/stdlib.h"
#include "hardware/adc.h"

#define TEST 0

// OTP range
#define OTP_MIN 100000
#define OTP_MAX 999999
#define OTP_MAXLEN 10

char* generate_code(uint16_t raw, char* secret_pin) {
	unsigned int seed = raw + atoi(secret_pin);
	srand(seed);
	int otp = OTP_MIN + rand() % (OTP_MAX + 1 - OTP_MIN); // Somehow, this is how you get a random int between OTP_MIN and OTP_MAX included
	char *code = malloc(OTP_MAXLEN * sizeof(char));
	itoa(otp, code, 10);
	return code;
}

// Core 0 Main Code
int main(void){
    stdio_init_all();
    char *secret_pin = "324092";
    // Configure ADC
    adc_init();
    adc_set_temp_sensor_enabled(true);
    adc_select_input(4); // Select adc for board temp

    while(1) {
		uint16_t raw;
		if(TEST) {
			raw = adc_read();
			char * code = generate_code(raw, secret_pin);
			float temp = 27 - ((raw * (3.3f / (1<<12))) -0.706)/0.001721;
			printf("temp = %f\n", temp);
			printf("raw = %d\n", raw);
			printf("code : %s\n", code);
		} else {
			for(float temp = -35; temp<-30; temp+=0.01) {
				raw = (0.706 + (27 - temp) * 0.001721) /(3.3f / (1<<12));
				char * code = generate_code(raw, secret_pin);
				printf("temp = %f | ", temp);
				printf("raw = %d | ", raw);
				printf("code : %s\n", code);
			}
		}
		sleep_ms(1000);
    }
}
```

## Traces sur le port série

```bash
temp = -35.000000 | raw = 1008 | code : 579843
temp = -34.990002 | raw = 1008 | code : 579843
temp = -34.980003 | raw = 1008 | code : 579843
temp = -34.970005 | raw = 1008 | code : 579843
temp = -34.960007 | raw = 1008 | code : 579843
temp = -34.950008 | raw = 1008 | code : 579843
temp = -34.940010 | raw = 1008 | code : 579843
temp = -34.930012 | raw = 1008 | code : 579843
temp = -34.920013 | raw = 1008 | code : 579843
temp = -34.910015 | raw = 1008 | code : 579843
temp = -34.900017 | raw = 1008 | code : 579843
temp = -34.890018 | raw = 1008 | code : 579843
temp = -34.880020 | raw = 1008 | code : 579843
temp = -34.870022 | raw = 1008 | code : 579843
temp = -34.860023 | raw = 1008 | code : 579843
temp = -34.850025 | raw = 1008 | code : 579843
temp = -34.840027 | raw = 1008 | code : 579843
temp = -34.830029 | raw = 1008 | code : 579843
temp = -34.820030 | raw = 1008 | code : 579843
temp = -34.810032 | raw = 1008 | code : 579843
temp = -34.800034 | raw = 1008 | code : 579843
temp = -34.790035 | raw = 1008 | code : 579843
temp = -34.780037 | raw = 1008 | code : 579843
temp = -34.770039 | raw = 1008 | code : 579843
temp = -34.760040 | raw = 1008 | code : 579843
temp = -34.750042 | raw = 1008 | code : 579843
temp = -34.740044 | raw = 1008 | code : 579843
temp = -34.730045 | raw = 1008 | code : 579843
temp = -34.720047 | raw = 1008 | code : 579843
temp = -34.710049 | raw = 1008 | code : 579843
temp = -34.700050 | raw = 1008 | code : 579843
temp = -34.690052 | raw = 1008 | code : 579843
temp = -34.680054 | raw = 1008 | code : 579843
temp = -34.670055 | raw = 1008 | code : 579843
temp = -34.660057 | raw = 1008 | code : 579843
temp = -34.650059 | raw = 1007 | code : 297558
temp = -34.640060 | raw = 1007 | code : 297558
temp = -34.630062 | raw = 1007 | code : 297558
temp = -34.620064 | raw = 1007 | code : 297558
temp = -34.610065 | raw = 1007 | code : 297558
temp = -34.600067 | raw = 1007 | code : 297558
temp = -34.590069 | raw = 1007 | code : 297558
temp = -34.580070 | raw = 1007 | code : 297558
temp = -34.570072 | raw = 1007 | code : 297558
temp = -34.560074 | raw = 1007 | code : 297558
temp = -34.550076 | raw = 1007 | code : 297558
temp = -34.540077 | raw = 1007 | code : 297558
temp = -34.530079 | raw = 1007 | code : 297558
temp = -34.520081 | raw = 1007 | code : 297558
temp = -34.510082 | raw = 1007 | code : 297558
temp = -34.500084 | raw = 1007 | code : 297558
temp = -34.490086 | raw = 1007 | code : 297558
temp = -34.480087 | raw = 1007 | code : 297558
temp = -34.470089 | raw = 1007 | code : 297558
temp = -34.460091 | raw = 1007 | code : 297558
temp = -34.450092 | raw = 1007 | code : 297558
temp = -34.440094 | raw = 1007 | code : 297558
temp = -34.430096 | raw = 1007 | code : 297558
temp = -34.420097 | raw = 1007 | code : 297558
temp = -34.410099 | raw = 1007 | code : 297558
temp = -34.400101 | raw = 1007 | code : 297558
temp = -34.390102 | raw = 1007 | code : 297558
temp = -34.380104 | raw = 1007 | code : 297558
temp = -34.370106 | raw = 1007 | code : 297558
temp = -34.360107 | raw = 1007 | code : 297558
temp = -34.350109 | raw = 1007 | code : 297558
temp = -34.340111 | raw = 1007 | code : 297558
temp = -34.330112 | raw = 1007 | code : 297558
temp = -34.320114 | raw = 1007 | code : 297558
temp = -34.310116 | raw = 1007 | code : 297558
temp = -34.300117 | raw = 1007 | code : 297558
temp = -34.290119 | raw = 1007 | code : 297558
temp = -34.280121 | raw = 1007 | code : 297558
temp = -34.270123 | raw = 1007 | code : 297558
temp = -34.260124 | raw = 1007 | code : 297558
temp = -34.250126 | raw = 1007 | code : 297558
temp = -34.240128 | raw = 1007 | code : 297558
temp = -34.230129 | raw = 1007 | code : 297558
temp = -34.220131 | raw = 1007 | code : 297558
temp = -34.210133 | raw = 1007 | code : 297558
temp = -34.200134 | raw = 1007 | code : 297558
temp = -34.190136 | raw = 1007 | code : 297558
temp = -34.180138 | raw = 1006 | code : 915273
temp = -34.170139 | raw = 1006 | code : 915273
temp = -34.160141 | raw = 1006 | code : 915273
temp = -34.150143 | raw = 1006 | code : 915273
temp = -34.140144 | raw = 1006 | code : 915273
temp = -34.130146 | raw = 1006 | code : 915273
temp = -34.120148 | raw = 1006 | code : 915273
temp = -34.110149 | raw = 1006 | code : 915273
temp = -34.100151 | raw = 1006 | code : 915273
temp = -34.090153 | raw = 1006 | code : 915273
temp = -34.080154 | raw = 1006 | code : 915273
temp = -34.070156 | raw = 1006 | code : 915273
temp = -34.060158 | raw = 1006 | code : 915273
temp = -34.050159 | raw = 1006 | code : 915273
temp = -34.040161 | raw = 1006 | code : 915273
temp = -34.030163 | raw = 1006 | code : 915273
temp = -34.020164 | raw = 1006 | code : 915273
temp = -34.010166 | raw = 1006 | code : 915273
temp = -34.000168 | raw = 1006 | code : 915273
temp = -33.990170 | raw = 1006 | code : 915273
temp = -33.980171 | raw = 1006 | code : 915273
temp = -33.970173 | raw = 1006 | code : 915273
temp = -33.960175 | raw = 1006 | code : 915273
temp = -33.950176 | raw = 1006 | code : 915273
temp = -33.940178 | raw = 1006 | code : 915273
temp = -33.930180 | raw = 1006 | code : 915273
temp = -33.920181 | raw = 1006 | code : 915273
temp = -33.910183 | raw = 1006 | code : 915273
temp = -33.900185 | raw = 1006 | code : 915273
temp = -33.890186 | raw = 1006 | code : 915273
temp = -33.880188 | raw = 1006 | code : 915273
temp = -33.870190 | raw = 1006 | code : 915273
temp = -33.860191 | raw = 1006 | code : 915273
temp = -33.850193 | raw = 1006 | code : 915273
temp = -33.840195 | raw = 1006 | code : 915273
temp = -33.830196 | raw = 1006 | code : 915273
temp = -33.820198 | raw = 1006 | code : 915273
temp = -33.810200 | raw = 1006 | code : 915273
temp = -33.800201 | raw = 1006 | code : 915273
temp = -33.790203 | raw = 1006 | code : 915273
temp = -33.780205 | raw = 1006 | code : 915273
temp = -33.770206 | raw = 1006 | code : 915273
temp = -33.760208 | raw = 1006 | code : 915273
temp = -33.750210 | raw = 1006 | code : 915273
temp = -33.740211 | raw = 1006 | code : 915273
temp = -33.730213 | raw = 1006 | code : 915273
temp = -33.720215 | raw = 1006 | code : 915273
temp = -33.710217 | raw = 1005 | code : 549339
temp = -33.700218 | raw = 1005 | code : 549339
temp = -33.690220 | raw = 1005 | code : 549339
temp = -33.680222 | raw = 1005 | code : 549339
temp = -33.670223 | raw = 1005 | code : 549339
temp = -33.660225 | raw = 1005 | code : 549339
temp = -33.650227 | raw = 1005 | code : 549339
temp = -33.640228 | raw = 1005 | code : 549339
temp = -33.630230 | raw = 1005 | code : 549339
temp = -33.620232 | raw = 1005 | code : 549339
temp = -33.610233 | raw = 1005 | code : 549339
temp = -33.600235 | raw = 1005 | code : 549339
temp = -33.590237 | raw = 1005 | code : 549339
temp = -33.580238 | raw = 1005 | code : 549339
temp = -33.570240 | raw = 1005 | code : 549339
temp = -33.560242 | raw = 1005 | code : 549339
temp = -33.550243 | raw = 1005 | code : 549339
temp = -33.540245 | raw = 1005 | code : 549339
temp = -33.530247 | raw = 1005 | code : 549339
temp = -33.520248 | raw = 1005 | code : 549339
temp = -33.510250 | raw = 1005 | code : 549339
temp = -33.500252 | raw = 1005 | code : 549339
temp = -33.490253 | raw = 1005 | code : 549339
temp = -33.480255 | raw = 1005 | code : 549339
temp = -33.470257 | raw = 1005 | code : 549339
temp = -33.460258 | raw = 1005 | code : 549339
temp = -33.450260 | raw = 1005 | code : 549339
temp = -33.440262 | raw = 1005 | code : 549339
temp = -33.430264 | raw = 1005 | code : 549339
temp = -33.420265 | raw = 1005 | code : 549339
temp = -33.410267 | raw = 1005 | code : 549339
temp = -33.400269 | raw = 1005 | code : 549339
temp = -33.390270 | raw = 1005 | code : 549339
temp = -33.380272 | raw = 1005 | code : 549339
temp = -33.370274 | raw = 1005 | code : 549339
temp = -33.360275 | raw = 1005 | code : 549339
temp = -33.350277 | raw = 1005 | code : 549339
temp = -33.340279 | raw = 1005 | code : 549339
temp = -33.330280 | raw = 1005 | code : 549339
temp = -33.320282 | raw = 1005 | code : 549339
temp = -33.310284 | raw = 1005 | code : 549339
temp = -33.300285 | raw = 1005 | code : 549339
temp = -33.290287 | raw = 1005 | code : 549339
temp = -33.280289 | raw = 1005 | code : 549339
temp = -33.270290 | raw = 1005 | code : 549339
temp = -33.260292 | raw = 1005 | code : 549339
temp = -33.250294 | raw = 1004 | code : 267054
temp = -33.240295 | raw = 1004 | code : 267054
temp = -33.230297 | raw = 1004 | code : 267054
temp = -33.220299 | raw = 1004 | code : 267054
temp = -33.210300 | raw = 1004 | code : 267054
temp = -33.200302 | raw = 1004 | code : 267054
temp = -33.190304 | raw = 1004 | code : 267054
temp = -33.180305 | raw = 1004 | code : 267054
temp = -33.170307 | raw = 1004 | code : 267054
temp = -33.160309 | raw = 1004 | code : 267054
temp = -33.150311 | raw = 1004 | code : 267054
temp = -33.140312 | raw = 1004 | code : 267054
temp = -33.130314 | raw = 1004 | code : 267054
temp = -33.120316 | raw = 1004 | code : 267054
temp = -33.110317 | raw = 1004 | code : 267054
temp = -33.100319 | raw = 1004 | code : 267054
temp = -33.090321 | raw = 1004 | code : 267054
temp = -33.080322 | raw = 1004 | code : 267054
temp = -33.070324 | raw = 1004 | code : 267054
temp = -33.060326 | raw = 1004 | code : 267054
temp = -33.050327 | raw = 1004 | code : 267054
temp = -33.040329 | raw = 1004 | code : 267054
temp = -33.030331 | raw = 1004 | code : 267054
temp = -33.020332 | raw = 1004 | code : 267054
temp = -33.010334 | raw = 1004 | code : 267054
temp = -33.000336 | raw = 1004 | code : 267054
temp = -32.990337 | raw = 1004 | code : 267054
temp = -32.980339 | raw = 1004 | code : 267054
temp = -32.970341 | raw = 1004 | code : 267054
temp = -32.960342 | raw = 1004 | code : 267054
temp = -32.950344 | raw = 1004 | code : 267054
temp = -32.940346 | raw = 1004 | code : 267054
temp = -32.930347 | raw = 1004 | code : 267054
temp = -32.920349 | raw = 1004 | code : 267054
temp = -32.910351 | raw = 1004 | code : 267054
temp = -32.900352 | raw = 1004 | code : 267054
temp = -32.890354 | raw = 1004 | code : 267054
temp = -32.880356 | raw = 1004 | code : 267054
temp = -32.870358 | raw = 1004 | code : 267054
temp = -32.860359 | raw = 1004 | code : 267054
temp = -32.850361 | raw = 1004 | code : 267054
temp = -32.840363 | raw = 1004 | code : 267054
temp = -32.830364 | raw = 1004 | code : 267054
temp = -32.820366 | raw = 1004 | code : 267054
temp = -32.810368 | raw = 1004 | code : 267054
temp = -32.800369 | raw = 1004 | code : 267054
temp = -32.790371 | raw = 1004 | code : 267054
temp = -32.780373 | raw = 1003 | code : 884769
temp = -32.770374 | raw = 1003 | code : 884769
temp = -32.760376 | raw = 1003 | code : 884769
temp = -32.750378 | raw = 1003 | code : 884769
temp = -32.740379 | raw = 1003 | code : 884769
temp = -32.730381 | raw = 1003 | code : 884769
temp = -32.720383 | raw = 1003 | code : 884769
temp = -32.710384 | raw = 1003 | code : 884769
temp = -32.700386 | raw = 1003 | code : 884769
temp = -32.690388 | raw = 1003 | code : 884769
temp = -32.680389 | raw = 1003 | code : 884769
temp = -32.670391 | raw = 1003 | code : 884769
temp = -32.660393 | raw = 1003 | code : 884769
temp = -32.650394 | raw = 1003 | code : 884769
temp = -32.640396 | raw = 1003 | code : 884769
temp = -32.630398 | raw = 1003 | code : 884769
temp = -32.620399 | raw = 1003 | code : 884769
temp = -32.610401 | raw = 1003 | code : 884769
temp = -32.600403 | raw = 1003 | code : 884769
temp = -32.590405 | raw = 1003 | code : 884769
temp = -32.580406 | raw = 1003 | code : 884769
temp = -32.570408 | raw = 1003 | code : 884769
temp = -32.560410 | raw = 1003 | code : 884769
temp = -32.550411 | raw = 1003 | code : 884769
temp = -32.540413 | raw = 1003 | code : 884769
temp = -32.530415 | raw = 1003 | code : 884769
temp = -32.520416 | raw = 1003 | code : 884769
temp = -32.510418 | raw = 1003 | code : 884769
temp = -32.500420 | raw = 1003 | code : 884769
temp = -32.490421 | raw = 1003 | code : 884769
temp = -32.480423 | raw = 1003 | code : 884769
temp = -32.470425 | raw = 1003 | code : 884769
temp = -32.460426 | raw = 1003 | code : 884769
temp = -32.450428 | raw = 1003 | code : 884769
temp = -32.440430 | raw = 1003 | code : 884769
temp = -32.430431 | raw = 1003 | code : 884769
temp = -32.420433 | raw = 1003 | code : 884769
temp = -32.410435 | raw = 1003 | code : 884769
temp = -32.400436 | raw = 1003 | code : 884769
temp = -32.390438 | raw = 1003 | code : 884769
temp = -32.380440 | raw = 1003 | code : 884769
temp = -32.370441 | raw = 1003 | code : 884769
temp = -32.360443 | raw = 1003 | code : 884769
temp = -32.350445 | raw = 1003 | code : 884769
temp = -32.340446 | raw = 1003 | code : 884769
temp = -32.330448 | raw = 1003 | code : 884769
temp = -32.320450 | raw = 1003 | code : 884769
temp = -32.310452 | raw = 1002 | code : 518835
temp = -32.300453 | raw = 1002 | code : 518835
temp = -32.290455 | raw = 1002 | code : 518835
temp = -32.280457 | raw = 1002 | code : 518835
temp = -32.270458 | raw = 1002 | code : 518835
temp = -32.260460 | raw = 1002 | code : 518835
temp = -32.250462 | raw = 1002 | code : 518835
temp = -32.240463 | raw = 1002 | code : 518835
temp = -32.230465 | raw = 1002 | code : 518835
temp = -32.220467 | raw = 1002 | code : 518835
temp = -32.210468 | raw = 1002 | code : 518835
temp = -32.200470 | raw = 1002 | code : 518835
temp = -32.190472 | raw = 1002 | code : 518835
temp = -32.180473 | raw = 1002 | code : 518835
temp = -32.170475 | raw = 1002 | code : 518835
temp = -32.160477 | raw = 1002 | code : 518835
temp = -32.150478 | raw = 1002 | code : 518835
temp = -32.140480 | raw = 1002 | code : 518835
temp = -32.130482 | raw = 1002 | code : 518835
temp = -32.120483 | raw = 1002 | code : 518835
temp = -32.110485 | raw = 1002 | code : 518835
temp = -32.100487 | raw = 1002 | code : 518835
temp = -32.090488 | raw = 1002 | code : 518835
temp = -32.080490 | raw = 1002 | code : 518835
temp = -32.070492 | raw = 1002 | code : 518835
temp = -32.060493 | raw = 1002 | code : 518835
temp = -32.050495 | raw = 1002 | code : 518835
temp = -32.040497 | raw = 1002 | code : 518835
temp = -32.030499 | raw = 1002 | code : 518835
temp = -32.020500 | raw = 1002 | code : 518835
temp = -32.010502 | raw = 1002 | code : 518835
temp = -32.000504 | raw = 1002 | code : 518835
temp = -31.990503 | raw = 1002 | code : 518835
temp = -31.980503 | raw = 1002 | code : 518835
temp = -31.970503 | raw = 1002 | code : 518835
temp = -31.960503 | raw = 1002 | code : 518835
temp = -31.950502 | raw = 1002 | code : 518835
temp = -31.940502 | raw = 1002 | code : 518835
temp = -31.930502 | raw = 1002 | code : 518835
temp = -31.920502 | raw = 1002 | code : 518835
temp = -31.910501 | raw = 1002 | code : 518835
temp = -31.900501 | raw = 1002 | code : 518835
temp = -31.890501 | raw = 1002 | code : 518835
temp = -31.880501 | raw = 1002 | code : 518835
temp = -31.870501 | raw = 1002 | code : 518835
temp = -31.860500 | raw = 1002 | code : 518835
temp = -31.850500 | raw = 1002 | code : 518835
temp = -31.840500 | raw = 1001 | code : 236550
temp = -31.830500 | raw = 1001 | code : 236550
temp = -31.820499 | raw = 1001 | code : 236550
temp = -31.810499 | raw = 1001 | code : 236550
temp = -31.800499 | raw = 1001 | code : 236550
temp = -31.790499 | raw = 1001 | code : 236550
temp = -31.780499 | raw = 1001 | code : 236550
temp = -31.770498 | raw = 1001 | code : 236550
temp = -31.760498 | raw = 1001 | code : 236550
temp = -31.750498 | raw = 1001 | code : 236550
temp = -31.740498 | raw = 1001 | code : 236550
temp = -31.730497 | raw = 1001 | code : 236550
temp = -31.720497 | raw = 1001 | code : 236550
temp = -31.710497 | raw = 1001 | code : 236550
temp = -31.700497 | raw = 1001 | code : 236550
temp = -31.690496 | raw = 1001 | code : 236550
temp = -31.680496 | raw = 1001 | code : 236550
temp = -31.670496 | raw = 1001 | code : 236550
temp = -31.660496 | raw = 1001 | code : 236550
temp = -31.650496 | raw = 1001 | code : 236550
temp = -31.640495 | raw = 1001 | code : 236550
temp = -31.630495 | raw = 1001 | code : 236550
temp = -31.620495 | raw = 1001 | code : 236550
temp = -31.610495 | raw = 1001 | code : 236550
temp = -31.600494 | raw = 1001 | code : 236550
temp = -31.590494 | raw = 1001 | code : 236550
temp = -31.580494 | raw = 1001 | code : 236550
temp = -31.570494 | raw = 1001 | code : 236550
temp = -31.560493 | raw = 1001 | code : 236550
temp = -31.550493 | raw = 1001 | code : 236550
temp = -31.540493 | raw = 1001 | code : 236550
temp = -31.530493 | raw = 1001 | code : 236550
temp = -31.520493 | raw = 1001 | code : 236550
temp = -31.510492 | raw = 1001 | code : 236550
temp = -31.500492 | raw = 1001 | code : 236550
temp = -31.490492 | raw = 1001 | code : 236550
temp = -31.480492 | raw = 1001 | code : 236550
temp = -31.470491 | raw = 1001 | code : 236550
temp = -31.460491 | raw = 1001 | code : 236550
temp = -31.450491 | raw = 1001 | code : 236550
temp = -31.440491 | raw = 1001 | code : 236550
temp = -31.430490 | raw = 1001 | code : 236550
temp = -31.420490 | raw = 1001 | code : 236550
temp = -31.410490 | raw = 1001 | code : 236550
temp = -31.400490 | raw = 1001 | code : 236550
temp = -31.390490 | raw = 1001 | code : 236550
temp = -31.380489 | raw = 1001 | code : 236550
temp = -31.370489 | raw = 1000 | code : 854265
temp = -31.360489 | raw = 1000 | code : 854265
temp = -31.350489 | raw = 1000 | code : 854265
temp = -31.340488 | raw = 1000 | code : 854265
temp = -31.330488 | raw = 1000 | code : 854265
temp = -31.320488 | raw = 1000 | code : 854265
temp = -31.310488 | raw = 1000 | code : 854265
temp = -31.300488 | raw = 1000 | code : 854265
temp = -31.290487 | raw = 1000 | code : 854265
temp = -31.280487 | raw = 1000 | code : 854265
temp = -31.270487 | raw = 1000 | code : 854265
temp = -31.260487 | raw = 1000 | code : 854265
temp = -31.250486 | raw = 1000 | code : 854265
temp = -31.240486 | raw = 1000 | code : 854265
temp = -31.230486 | raw = 1000 | code : 854265
temp = -31.220486 | raw = 1000 | code : 854265
temp = -31.210485 | raw = 1000 | code : 854265
temp = -31.200485 | raw = 1000 | code : 854265
temp = -31.190485 | raw = 1000 | code : 854265
temp = -31.180485 | raw = 1000 | code : 854265
temp = -31.170485 | raw = 1000 | code : 854265
temp = -31.160484 | raw = 1000 | code : 854265
temp = -31.150484 | raw = 1000 | code : 854265
temp = -31.140484 | raw = 1000 | code : 854265
temp = -31.130484 | raw = 1000 | code : 854265
temp = -31.120483 | raw = 1000 | code : 854265
temp = -31.110483 | raw = 1000 | code : 854265
temp = -31.100483 | raw = 1000 | code : 854265
temp = -31.090483 | raw = 1000 | code : 854265
temp = -31.080482 | raw = 1000 | code : 854265
temp = -31.070482 | raw = 1000 | code : 854265
temp = -31.060482 | raw = 1000 | code : 854265
temp = -31.050482 | raw = 1000 | code : 854265
temp = -31.040482 | raw = 1000 | code : 854265
temp = -31.030481 | raw = 1000 | code : 854265
temp = -31.020481 | raw = 1000 | code : 854265
temp = -31.010481 | raw = 1000 | code : 854265
temp = -31.000481 | raw = 1000 | code : 854265
temp = -30.990480 | raw = 1000 | code : 854265
temp = -30.980480 | raw = 1000 | code : 854265
temp = -30.970480 | raw = 1000 | code : 854265
temp = -30.960480 | raw = 1000 | code : 854265
temp = -30.950480 | raw = 1000 | code : 854265
temp = -30.940479 | raw = 1000 | code : 854265
temp = -30.930479 | raw = 1000 | code : 854265
temp = -30.920479 | raw = 1000 | code : 854265
temp = -30.910479 | raw = 999 | code : 488331
temp = -30.900478 | raw = 999 | code : 488331
temp = -30.890478 | raw = 999 | code : 488331
temp = -30.880478 | raw = 999 | code : 488331
temp = -30.870478 | raw = 999 | code : 488331
temp = -30.860477 | raw = 999 | code : 488331
temp = -30.850477 | raw = 999 | code : 488331
temp = -30.840477 | raw = 999 | code : 488331
temp = -30.830477 | raw = 999 | code : 488331
temp = -30.820477 | raw = 999 | code : 488331
temp = -30.810476 | raw = 999 | code : 488331
temp = -30.800476 | raw = 999 | code : 488331
temp = -30.790476 | raw = 999 | code : 488331
temp = -30.780476 | raw = 999 | code : 488331
temp = -30.770475 | raw = 999 | code : 488331
temp = -30.760475 | raw = 999 | code : 488331
temp = -30.750475 | raw = 999 | code : 488331
temp = -30.740475 | raw = 999 | code : 488331
temp = -30.730474 | raw = 999 | code : 488331
temp = -30.720474 | raw = 999 | code : 488331
temp = -30.710474 | raw = 999 | code : 488331
temp = -30.700474 | raw = 999 | code : 488331
temp = -30.690474 | raw = 999 | code : 488331
temp = -30.680473 | raw = 999 | code : 488331
temp = -30.670473 | raw = 999 | code : 488331
temp = -30.660473 | raw = 999 | code : 488331
temp = -30.650473 | raw = 999 | code : 488331
temp = -30.640472 | raw = 999 | code : 488331
temp = -30.630472 | raw = 999 | code : 488331
temp = -30.620472 | raw = 999 | code : 488331
temp = -30.610472 | raw = 999 | code : 488331
temp = -30.600471 | raw = 999 | code : 488331
temp = -30.590471 | raw = 999 | code : 488331
temp = -30.580471 | raw = 999 | code : 488331
temp = -30.570471 | raw = 999 | code : 488331
temp = -30.560471 | raw = 999 | code : 488331
temp = -30.550470 | raw = 999 | code : 488331
temp = -30.540470 | raw = 999 | code : 488331
temp = -30.530470 | raw = 999 | code : 488331
temp = -30.520470 | raw = 999 | code : 488331
temp = -30.510469 | raw = 999 | code : 488331
temp = -30.500469 | raw = 999 | code : 488331
temp = -30.490469 | raw = 999 | code : 488331
temp = -30.480469 | raw = 999 | code : 488331
temp = -30.470469 | raw = 999 | code : 488331
temp = -30.460468 | raw = 999 | code : 488331
temp = -30.450468 | raw = 999 | code : 488331
temp = -30.440468 | raw = 998 | code : 206046
temp = -30.430468 | raw = 998 | code : 206046
temp = -30.420467 | raw = 998 | code : 206046
temp = -30.410467 | raw = 998 | code : 206046
temp = -30.400467 | raw = 998 | code : 206046
temp = -30.390467 | raw = 998 | code : 206046
temp = -30.380466 | raw = 998 | code : 206046
temp = -30.370466 | raw = 998 | code : 206046
temp = -30.360466 | raw = 998 | code : 206046
temp = -30.350466 | raw = 998 | code : 206046
temp = -30.340466 | raw = 998 | code : 206046
temp = -30.330465 | raw = 998 | code : 206046
temp = -30.320465 | raw = 998 | code : 206046
temp = -30.310465 | raw = 998 | code : 206046
temp = -30.300465 | raw = 998 | code : 206046
temp = -30.290464 | raw = 998 | code : 206046
temp = -30.280464 | raw = 998 | code : 206046
temp = -30.270464 | raw = 998 | code : 206046
temp = -30.260464 | raw = 998 | code : 206046
temp = -30.250463 | raw = 998 | code : 206046
temp = -30.240463 | raw = 998 | code : 206046
temp = -30.230463 | raw = 998 | code : 206046
temp = -30.220463 | raw = 998 | code : 206046
temp = -30.210463 | raw = 998 | code : 206046
temp = -30.200462 | raw = 998 | code : 206046
temp = -30.190462 | raw = 998 | code : 206046
temp = -30.180462 | raw = 998 | code : 206046
temp = -30.170462 | raw = 998 | code : 206046
temp = -30.160461 | raw = 998 | code : 206046
temp = -30.150461 | raw = 998 | code : 206046
temp = -30.140461 | raw = 998 | code : 206046
temp = -30.130461 | raw = 998 | code : 206046
temp = -30.120461 | raw = 998 | code : 206046
temp = -30.110460 | raw = 998 | code : 206046
temp = -30.100460 | raw = 998 | code : 206046
temp = -30.090460 | raw = 998 | code : 206046
temp = -30.080460 | raw = 998 | code : 206046
temp = -30.070459 | raw = 998 | code : 206046
temp = -30.060459 | raw = 998 | code : 206046
temp = -30.050459 | raw = 998 | code : 206046
temp = -30.040459 | raw = 998 | code : 206046
temp = -30.030458 | raw = 998 | code : 206046
temp = -30.020458 | raw = 998 | code : 206046
temp = -30.010458 | raw = 998 | code : 206046
temp = -30.000458 | raw = 998 | code : 206046
```