# FRITADEIRA ELÉTRICA
Olá, meu nome é Lailton Martins Canabrava e neste projeto é efetuada a simulação de uma fritadeira elétrica que possui funções como controlar a temperatura e definir o tempo.

#include "config.h"
#include "bits.h"
#include "pic18f4520.h"
#include "ssd.h"
#include "keypad.h"
#include "timer.h"
#include "lcd.h"
#include "pwm1.h"



void main(void) {

    unsigned char botao, temperatura, i, LCD = 1, vel = 1;
    unsigned char ventoinha = 0, timer = 0;
    unsigned long SSD = 0;

    lcdInit();
    kpInit();
    pwmInit();
    ssdInit();
    timerInit();

    for (;;) {
        timerReset(10000);
        kpDebounce();

        if (timer == 1) {
            ssdUpdate();
            ssdDigit(((SSD / 60000) % 6), 0);
            ssdDigit(((SSD / 6000) % 10), 1);
            ssdDigit(((SSD / 1000) % 6), 2);
            ssdDigit(((SSD / 100) % 10), 3);
            SSD--;

            if (SSD == 0) {
                pwmSet1(0);
                timer = 0;
            }
        }

        if (temperatura != kpRead()) {
            temperatura = kpRead();
            botao = kpRead();

            if (bitTst(botao, 3)) {
                if ((ventoinha % 2) == 0) {              
                    pwmSet1(32);
                    vel = 1;
                } else if ((ventoinha % 2) != 0) {        
                    pwmSet1(0);
                    vel = 1;
                }
                ventoinha++;
            }

            if (bitTst(botao, 7)) {
                if (vel == 1) { 
                    ventoinha = 1;
                    pwmSet1(0);
                    pwmSet1(32);
                    vel++; 
                } else if (vel == 2) { 
                    ventoinha = 1;
                    pwmSet1(0);
                    pwmSet1(64);
                    vel++; 
                } else if (vel == 3) {
                    ventoinha = 1;
                    pwmSet1(0);
                    pwmSet1(96);
                    vel = 1; 
                }
            }

            if (bitTst(botao, 6)) { 
                timer = 1;
                SSD = 3000;
            }
            if (bitTst(botao, 5)) { 
                timer = 1;
                SSD = 6000;
            }
            
            if (bitTst(botao, 2)) { // Tecla 7 = Tela 1
                lcdCommand(0x01);
                char LCD1[32] = "Batata Frita\nTemperatura";
                for (i = 0; i < 32; i++) {
                    if (LCD1[i] == '\n') {
                        lcdCommand(0xC0);
                        continue;
                    }
                    lcdData(LCD1[i]);
                }
            }
            if (bitTst(botao, 1)) { // Tecla * = Tela 2
                lcdCommand(0x01);
                char LCD2[32] = "1 minuto\n30 segundos";
                for (i = 0; i < 32; i++) {
                    if (LCD2[i] == '\n') {
                        lcdCommand(0xC0);
                        continue;
                    }
                    lcdData(LCD2[i]);
                }
            }
            
                }
            
        }
        timerWait();
    }
