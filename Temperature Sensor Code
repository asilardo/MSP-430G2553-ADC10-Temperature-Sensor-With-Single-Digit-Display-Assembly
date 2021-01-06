;-------------------------------------------------------------------------------
;       Description of code:
;       The goal of this program is to use the internal temperature sensor to
;       read the temperature once every second. On the display, the least 
;       significant nybble of the hexadecimal temperature value gets displayed.
;       A new value is updated on the display every second. 
;
;       Written by: Antonio Ilardo
;       Date:       November 16, 2020
;       Last Revision:  1.1 (November 13, 2020 formatted header description)
;                 1.2 November 14, 2020, used the
;                   DemoWithADC10TempSensorPolling.s43 to begin 
;                   working and analyzing the project. 
;                 1.3 November 15, 2020, Implemented display
;                 1.4 (November 16, 2020) Formatted code and added comments
;       ECE, Texas Tech University
;       Adapted from example code from Dr. M. Helm: 
;       DemoWithADC10TempSensorPolling.s43
;       Target: TI LaunchPad (MSP-EXP430GET version) development board with 
;           MSP430G2553 device installed
;       Assembler/IDE:  IAR Embedded Workbench 7.12 Kickstart version

;
;       HW I/O assignments:
;       P1.0    LED1    (Active HIGH)RED (not used in this example)
;       P1.1    not used
;       P1.2    not used
;       P1.3    PushButton (Active LOW) (internal Pullup Enabled) (not used)
;       P1.4    not used
;       P1.5    not used
;       P1.6    LED2    (Active HIGH)GREEN  (not used in this example
;       P1.7    not used
;
;       P2.0    not used
;       P2.1    not used
;       P2.2    not used
;       P2.3    not used
;       P2.4    not used
;       P2.5    not used
;       P2.6    not used
;       P2.7    not used
;
;
;*******************************************************************************
#include  "msp430g2553.h"
;-------------------------------------------------------------------------------
            ORG     0C000h
;-------------------------------------------------------------------------------
LONG_DELAY      EQU     65535  ; max 16 bit value (FFFFh)
;LONG_DELAY     EQU     600  ; max 16 bit value (FFFFh)
SHORT_DELAY     EQU     5000  ;

TIMER_A0_COUNT_1   EQU   2000    ; set count value for TimerA_0
;results in a 2 mS interrupt rate for updating each digit position in the
; display based on 1 MHz SMCLK/1 and counting 2000 of the 1 uS clock events
TIMER_A1_COUNT_1   EQU   50000    ; set count value for TimerA_1  
; results in a 100 mS basic interrupt rate based on 1 MHz SMCLK/2
; this will be a clock rate of one event per 2 uS, counting 50000 of those
; results in one interrupt every 100 mS from this timer


SEG_A         EQU     %00000001 ; Port pin position P2.0
SEG_B         EQU     %00000010 ; Port pin position P2.1
SEG_C         EQU     %00000100 ; Port pin position P2.2
SEG_D         EQU     %00001000 ; Port pin position P2.3
SEG_E         EQU     %00010000 ; Port pin position P2.4
SEG_F         EQU     %00100000 ; Port pin position P2.5
SEG_G         EQU     %01000000 ; Port pin position P2.6
SEG_DP        EQU     %10000000 ; Port pin position P2.7



SEG_PORT         EQU     P2OUT
PB_PORT          EQU     P1IN

ZERO            EQU     %00111111
ONE             EQU     %00000110
TWO             EQU     %01011011
THREE           EQU     %01001111
FOUR            EQU     %01100110
FIVE            EQU     %01101101
SIX             EQU     %01111100
SEVEN           EQU     %00000111
EIGHT           EQU     %01111111
NINE            EQU     %01100111
A               EQU     %01110111
B               EQU     %01111100
C1              EQU     %00111001                 
D               EQU     %01011110
E               EQU     %01111001
F               EQU     %01110001

;-------------------------------------------------------------------------------
; Definition of Variables
;-------------------------------------------------------------------------------
                ORG   0x0200     ; Start of RAM space (necessary statement)

CountMode       DW  0       ;Boolean flag value TRUE (1) when counting
DisplayValue    DW  0       ;Contains 100 mS count

;-------------------------------------------------------------------------------
            ORG     0xC000
;-------------------------------------------------------------------------------
RESET       mov.w   #0x0400,SP         ; Initialize stackpointer


StopWDT     mov.w   #WDTPW+WDTHOLD,&WDTCTL  ; Stop WDT
                                                  

SetupADC10  
            ; Has a 64x sample and hold time 
            ; implementation. This turns ADC10 on  
            mov.w   #REFON+SREF_1+ADC10SHT_3+ADC10ON,&ADC10CTL0  

            ; Sets the input channel to internal temperature 
            ; sensor. This will divide clock by 8
            mov.w   #INCH_10+ADC10DIV_7, &ADC10CTL1  


SetupP2              
           mov.b   #0xFF, &P2DIR ; all as outputs to drive 7-seg LED
                                                           
           bic.b    #0xC0, &P2SEL   ; Clears the P2SEL bits so that
                                        ; P2.6 and P2.7 function as GPIO pins
                                       
           mov.b    #0xBE, &P1REN    ;turn on the internal resistor for PB's
                                     ; and all other "input" mode pins
           mov.b    #0xBE, &P1OUT    ; set the resistors to Pullup mode                                        


SetupCalibratedClock      
; Clock setup (calibrated mode at 1 MHz)
; Use calibrated data for the DCO clock
; Set DCO to 1 MHz
          CLR.B &DCOCTL ; Lowest DCOx  and MODx settings are selected
          MOV.B &CALBC1_1MHZ,&BCSCTL1 ; Initialize range
          MOV.B &CALDCO_1MHZ,&DCOCTL ; Initialize DCO step + modulation      
       

SetupTimerA0
            ;TimerA_0: 2 mS intervals to update the 
            ;next digit of the multiplexed display
            
            ;Count value is loaded into the timer
            mov.w   #TIMER_A0_COUNT_1,&TA0CCR0 
            mov.w   #CCIE,&TA0CCTL0     ; Enable the timer interrupt

            mov.w   #TASSEL_2+ID_0+MC_1,&TA0CTL   ; SMCLK/1, up mode  
     

SetupTimerA1      
            ;TimerA_1: 00 mS intervals for the basic clock 
            ;counting rate (higher priority than TimerA_0)
            
            ;Count value is loaded into the timer 
            mov.w   #TIMER_A1_COUNT_1,&TA1CCR0 
            mov.w   #CCIE,&TA1CCTL0        ; Enable the timer interrupt

            mov.w   #TASSEL_2+ID_1+MC_1,&TA1CTL   ; SMCLK/2, up mode                
     

ClearRAMVariables  
            clr.b   &CountMode
            clr.b   &DisplayValue ; Clears the display 


EnableGeneralInterrupts
            bis.b #GIE,SR          ; General interrupts bit is enabled
       

Mainloop    
            bis.w   #ENC+ADC10SC,&ADC10CTL0 ; Sampling/conversion is intialized
 

TestInProgress    
            bit   #01h, &ADC10CTL1     ; Check the ADC10BUSY bit (LSbit)

            jne     TestInProgress     ; While still in progress, keep checking
            
            jmp     Mainloop              ;Repeat code
                                          
   
;-------------------------------------------------------------------------------
;           End of main code
;-------------------------------------------------------------------------------                                            
                                           
;-------------------------------------------------------------------------------
;           Subroutines
;-------------------------------------------------------------------------------

;-------------------------------------------------------------------------------
;  DisplayDigit
;  passed in - DisplayValue, CurrentDigitPos
;  returned - nothing
;  accomplishes - Writes next digit to the expansion bd display
;  uses: R15, global variable CurrentDigitPos, CurrentDigitValue
;-------------------------------------------------------------------------------
DisplayDigit
    push R15     ; save R15 since we use it here

    mov.w &DisplayValue, R15  ; make a copy
    and.w #0x000F, R15
   ; rra.w R15                     ; get the value into LSnibble position
   ; rra.w R15
   ; rra.w R15
    ;rra.w R15
    add.w #PatternTable, R15  ; R15 now points to correct Seg pattern to write
    mov.b @R15, SEG_PORT       ; set up the pattern to write

    pop R15     ; restore R15 before returning
    ret         ; return
;-------------------------------------------------------------------------------
;  end of WriteNextDigitToDisplay
;-------------------------------------------------------------------------------


;-------------------------------------------------------------------------------
; Delay_Long
;  passed in - nothing
;  returned - nothing
;  accomplishes - long delay
;  uses: R15
;-------------------------------------------------------------------------------
Delay_Long
    push R15     ; save R15 since we use it here
DelayTopL
    mov #LONG_DELAY, R15     ;load loop counter (R15) with Long Delay constant
Loop1Long
    dec R15                     ; decrement loop counter
    jnz Loop1Long               ; Zero yet?, no decrement again

    pop R15     ; restore R15 before returning
    ret         ; return
;-------------------------------------------------------------------------------
;  end of Delay_Long
;-------------------------------------------------------------------------------

;-------------------------------------------------------------------------------
;           End of all Subroutines
;-------------------------------------------------------------------------------

;-------------------------------------------------------------------------------
;          Interrupt Service Routines
;-------------------------------------------------------------------------------

TimerA1_ISR
     ;uses 100 mS implementation
     inc &CountMode 
     cmp #0xA, CountMode
     jeq ProgramStart
     reti ;end interrupt service routine
  
  
ProgramStart
      clr &CountMode
      mov.w &ADC10MEM, &DisplayValue     ;stores ADC result in Displayvalue
      reti ;end interrupt service routine


TimerA0_ISR
      ; Uses 2 mS implementation, resulting in display update every 8 mS
      call #DisplayDigit
      reti    ; return from interrupt

;-------------------------------------------------------------------------------
;          End of Interrupt Service Routines
;-------------------------------------------------------------------------------

;-------------------------------------------------------------------------------
;           Definition of Program Data Memory
;-------------------------------------------------------------------------------
            ORG   0xE000   ;memory allocation of program data

; Uses PatternTable as base address, then offset into table for
; a specific 7-seg number (pattern) to display
PatternTable  ; contains patterns for each 7-seg number
     DB  ZERO
     DB  ONE
     DB  TWO
     DB  THREE
     DB  FOUR
     DB  FIVE
     DB  SIX
     DB  SEVEN
     DB  EIGHT
     DB  NINE
     DB  A
     DB  B
     DB  C1
     DB  D
     DB  E
     DB  F
     

;-------------------------------------------------------------------------------
;           Interrupt Vectors
;-------------------------------------------------------------------------------

            ORG     0xFFFE                  ; MSP430 RESET Vector
            DW      RESET                   ; establishes the label RESET as
           
            ORG     0xFFF2                  ;TimerA_0 Vector
            DW      TimerA0_ISR             ;TimerA_0 Interrupt Service Routine
           
            ORG     0xFFFA                  ;TimerA_1 Vector
            DW      TimerA1_ISR             ;TimerA_1 Interrupt Service Routine

                                            ; the starting point
            END                             ; END code for the program
;-------------------------------------------------------------------------------
;-------------------------------------------------------------------------------
