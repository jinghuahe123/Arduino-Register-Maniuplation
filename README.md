# Arduino-Register-Maniuplation

## ATMEGA328P Digital Pin Manipulation

The ATMEGA328P has three ports: PORTB, PORTC and PORTD. Each of the pins are mapped to a certain port.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/12c58c22-0436-4ac1-83c7-f8e1c8172ebc" />

To set a pin as an output/input, set/clear the bit in the corresponding data direction register. The default pin setting (0) is an input. 

```
// e.g.
DDRB |= (1 << PB4); // Sets digital pin 12 as an output (Pin4 on PORTB).
DDRC &= ~(1 << PC1); // Sets A1 as an input (Pin1 on PORTC).
DDRD |= (1 << PD2); // Sets digital pin 2 as an output (Pin2 on PORTD).
```

To set the state of a pin, set/clear the corresponding bit in the port data register. The default setting (0) is off.

```
PORTB |= (1 << PB4); // Pulls digital pin 12 high (Pin4 on PORTB).
PORTC &= ~(1 << PC1); // Pulls A1 low (Pin1 on PORTC).
PORTD &= ~(1 << PD2); // Pulls digital pin 2 low (Pin2 on PORTD).
```

// more to add (analogread/digitalread/^=pin toggle)

## ATMEGA328P Basic PWM Configurations

| Arduino Pin | MCU Port/Pin | Timer   | OCR Register | DDR Register | PORT Register | TCCRxA Configuration (Non-Inverting PWM)         | TCCRxB Configuration (Prescaler=1)              | Frequency (Prescaler=1) |
|-------------|--------------|---------|--------------|--------------|---------------|--------------------------------------------------|-------------------------------------------------|-------------------------|
| 3           | PD3          | Timer2  | OCR2B        | DDRD         | PORTD         | (1<<COM2B1) \| (1<<WGM21) \| (1<<WGM20)          | (1<<WGM22) \| (1<<CS20)                         | 62.5 kHz                |
| 5           | PD5          | Timer0  | OCR0B        | DDRD         | PORTD         | (1<<COM0B1) \| (1<<WGM01) \| (1<<WGM00)          | (1<<WGM02) \| (1<<CS00)                         | 62.5 kHz                |
| 6           | PD6          | Timer0  | OCR0A        | DDRD         | PORTD         | (1<<COM0A1) \| (1<<WGM01) \| (1<<WGM00)          | (1<<WGM02) \| (1<<CS00)                         | 62.5 kHz                |
| 9           | PB1          | Timer1  | OCR1A        | DDRB         | PORTB         | (1<<COM1A1) \| (1<<WGM11) \| (1<<WGM10)          | (1<<WGM12) \| (1<<WGM13) \| (1<<CS10)           | 62.5 kHz                |
| 10          | PB2          | Timer1  | OCR1B        | DDRB         | PORTB         | (1<<COM1B1) \| (1<<WGM11) \| (1<<WGM10)          | (1<<WGM12) \| (1<<WGM13) \| (1<<CS10)           | 62.5 kHz                |
| 11          | PB3          | Timer2  | OCR2A        | DDRB         | PORTB         | (1<<COM2A1) \| (1<<WGM21) \| (1<<WGM20)          | (1<<WGM22) \| (1<<CS20)                         | 62.5 kHz                |

This configuration assumes FAST 8-bit Non-Inverting PWM mode. Timer 1 supports up to 16bit operation.

**Timer 0 PWM Setup Example: (Pins 5 & 6)**

```
TCCR0A = (1<<WGM01) | (1<<WGM00) |  // Mode 3
         (1<<COM0A1) | // Enable Pin 6 (PD6) for PWM
         (1<<COM0B1);  // Enable Pin 5 (PD5) for PWM

// Default is 64, this makes millis() go 64x faster!!
TCCR0B = (1<<CS00); // Prescaler=1

OCR0A = 0; // Controls Pin 6, sets to 0% duty cycle
OCR0B = 128; // Controls Pin 5, sets to 50% duty cycle
```

**Timer 1 PWM Setup Example: (Pins 9 & 10)**
```
TCCR1A = (1<<WGM11) | (1<<WGM10) |  // Mode 5 (part 1)
         (1<<COM1A1) | // Enable Pin 9 (PB1) for PWM
         (1<<COM1B1);  // Enable Pin 10 (PB2) for PWM

TCCR1B = (1 << WGM12) | // Mode 5 (part 2)
         (1 << CS10); // Prescaler 1

OCR1A = 0; // Controls Pin 9, sets to 0% duty cycle
OCR1B = 128; // Controlls Pin 10, sets to 50% duty cycle
```

**Timer 2 PWM Setup Example: (Pins 3 & 11)**
```
TCCR2A = (1 << WGM21) | (1 << WGM20) | // Mode 3
           (1 << COM2A1) | // Enable Pin 11 (PB3) for PWM
           (1 << COM2B1);  // Enable Pin 3 (PD3) for PWM

TCCR2B = (1 << CS20); // Prescaler 1

OCR2A = 0; // Controls Pin 11, sets to 0% duty cycle
OCR2B = 128; // Controls Pin 3, sets to 50% duty cycle

// Modify registers OCR2A and OCR2B from 0 to 255 for controlling PWM duty cycle. 
```

**Example Usage with Pin 11 (PB3):**
```
DDRB |= (1 << PB3); // Sets Pin 11 as output

// Configure Timer2 for PWM
TCCR2A = (1 << COM2A1) | (1 << WGM21) | (1 << WGM20);  // Non-inverting PWM, Mode 3 (Fast PWM 8-bit)
TCCR2B = (1 << CS20);                                  // Prescaler = 1 (no prescaling)

OCR2A = 64; // 25% duty cycle
```

## ATMEGA328P Variable PWM Frequencies

The frequency is calculated by:

$$f_{PWM} = \frac{F_{CPU}}{N \times (TOP+1)}$$

F_{CPU} = 16Mhz (usually), the Arduino clock

N = prescaler (1,8, 64, 256, 1024)

TOP = a chosen value between 0 and 65535 (For Timer0 and Timer2, this value is fixed at 255)

| TOP Value | Frequency | Example (Prescaler=1)                           |
| --------- | --------- | ----------------------------------------------- |
| Decreases | Increases | TOP = 255 -> 62.5kHz <br> TOP = 127 -> 125kHz   |
| Increases | Decreases | TOP = 255 -> 62.5kHz <br> TOP = 1023 -> 15.6kHz |

However, decreasing the TOP value decreases the resolution. You can either have a very high frequency, but a low resolution or a very high resolution but a low frequency. You cannot have both.

The Resolution (bits) is given by:

$$ Resolution = log_2(TOP+1)$$

Such that $TOP+1$ is the number of steps available and the step size is given as $1/(TOP+1)$ as a fraction of the period. 

### Prescaler Options

The Prescaler defines how much the system clock is defined by. It can be used on all three timers to adjust the PWM frequency. 

Caution: Changing the prescaler on Timer0 is not reccomended as it can affect system clock functions such as delay() and millis(). 

**Timer0 (8-bit) - TCCR0B [CS02, CS01, CS00]**

| CS02 | CS01 | CS00 | Prescaler | Description            |
| ---- | ---- | ---- | --------- | ---------------------- |
| 0    | 0    | 0    | Stopped   | No clock (timer off)   |
| 0    | 0    | 1    | 1         | No prescaling (16 MHz) |
| 0    | 1    | 0    | 8         | Clock ÷ 8              |
| 0    | 1    | 1    | 64        | Clock ÷ 64             |
| 1    | 0    | 0    | 256       | Clock ÷ 256            |
| 1    | 0    | 1    | 1024      | Clock ÷ 1024           |

**Timer1 (16-bit) - TCCR1B [CS12, CS11, CS10]**

| CS12 | CS11 | CS10 | Prescaler | Description            |
| ---- | ---- | ---- | --------- | ---------------------- |
| 0    | 0    | 0    | Stopped   | No clock (timer off)   |
| 0    | 0    | 1    | 1         | No prescaling (16 MHz) |
| 0    | 1    | 0    | 8         | Clock ÷ 8              |
| 0    | 1    | 1    | 64        | Clock ÷ 64             |
| 1    | 0    | 0    | 256       | Clock ÷ 256            |
| 1    | 0    | 1    | 1024      | Clock ÷ 1024           |

**Timer2 (8-bit) - TCCR2B [CS22, CS21, CS20]**

| CS22 | CS21 | CS20 | Prescaler | Description            |
| ---- | ---- | ---- | --------- | ---------------------- |
| 0    | 0    | 0    | Stopped   | No clock (timer off)   |
| 0    | 0    | 1    | 1         | No prescaling (16 MHz) |
| 0    | 1    | 0    | 8         | Clock ÷ 8              |
| 0    | 1    | 1    | 32        | Clock ÷ 32             |
| 1    | 0    | 0    | 64        | Clock ÷ 64             |
| 1    | 0    | 1    | 128       | Clock ÷ 128            |
| 1    | 1    | 0    | 256       | Clock ÷ 256            |
| 1    | 1    | 1    | 1024      | Clock ÷ 1024           |

*Note: Timer2 can be clocked externally (e.g. by a 32.768kHz crystal for precise RTC operations). This is why it supports all eight prescaler combinations.*

**Example Usage for Timer0 on Pin 5 (PD5) w/ Prescaler=256**

```
TCCR0A = (1<<WGM01) | (1<<WGM00) |  // Mode 3
         (1<<COM0B1);  // Enable Pin 5 (PD5) for PWM

TCCR0B = (1<<CS02); // Prescaler=256

OCR0B = 128; // Controls Pin 5, sets to 50% duty cycle
```

**Example Usage for Timer1 on Pin 9 (PB1) w/ Prescaler=64**

*TOP is not set in this example. Timer1 Emulates 8bit PWM.*

```
TCCR1A = (1<<WGM11) | (1<<WGM10) |  // Mode 5 (part 1)
         (1<<COM1A1); // Enable Pin 9 (PB1) for PWM

TCCR1B = (1 << WGM12) | // Mode 5 (part 2)
         (1 << CS10) | (1 << CS11); // Prescaler=64

OCR1A = 64; // Controls Pin 9, sets to 25% duty cycle
```

**Example Usage for Timer2 on Pin 3 (PD3) w/ Prescaler=128**
```
TCCR2A = (1 << WGM21) | (1 << WGM20) | // Mode 3
           (1 << COM2B1);  // Enable Pin 3 (PD3) for PWM

TCCR2B = (1 << CS22) | (1 << CS20); // Prescaler 128

OCR2B = 128; // Controls Pin 3, sets to 50% duty cycle
```

### Timer1 Variable TOP

Timer1 support variable TOP values in certain PWM modes. Unlike Timer0 and Timer1, whos TOP value cannot be changed without losing functionality of Pin 6 (PD6 / OCR0A) and Pin 11 (PB3 / OCR2A). This functionality is due to Timer1's support of the Input Capture Register (ICR1).

For this application, Mode 14 can be used to generate a high frequency PWM signal. The TOP value in this mode is set within the ICR1 register. 

**Example Usage with Pin 9 (PB1) @ 1MHz**

Target f = 1,000,000Hz @ N = 1 prescaler. 

TOP = (F_CPU / (N × f)) - 1 = (16,000,000 / (1 × 1,000,000)) - 1 = 15.

The resolution will be 4 bits (16 steps, 6.25% per step).

```
DDRB |= (1 << PB1); // Set Pin 9 as output

ICR1 = 15;  // Set TOP value in ICR1

TCCR1A = (1 << COM1A1) | // Non-inverting PWM on Pin 9
         (1 << WGM11); // Part 1 of WGM13:0=1110

TCCR1B = (1 << WGM12) | (1 << WGM13) | // Part 2 of WGM
         (1 << CS10); // Prescaler = 1

OCR1A = 8; // Set Duty Cycle to 50%
```

**Example Usage with Pin 9 (PB1) @ 50Hz**

e.g. generate a 50Hz signal for a servo (period of 20ms)

At N=1, TOP = (16e6/50)-1 = 319999, which is too large for a 16bit timer. Therefore, we use prescaler N = 64.

TOP = (16,000,000 / (64 x 50)) - 1 = 5000 - 1 = 4999.

The resolution will be ~12.3 bits (5000 steps, 0.02% per step).

```
DDRB |= (1 << PB1); // Set Pin 9 as output

ICR1 = 4999; // Set TOP value

TCCR1A = (1 << COM1A1) | (1 << WGM11);
TCCR1B = (1 << WGM12) | (1 << WGM13) |
         (1 << CS11) | (1 << CS10); // Prescaler 64

OCR1A = 375; // 50Hz signal @ 1.5ms/pulse = 7.5% of 20ms. 7.5% of 5000 = 375
```

**Example Usage with Pin 9 (PB1) @ 10bit Resolution (15.625kHz)**

TOP = 2^10 - 1 = 1023

Frequency = 16,000,000 / (1 × (1023 + 1)) = 15,625 Hz. @ N=1

```
DDRB |= (1 << PB1); 

ICR1 = 1023; 

TCCR1A = (1 << COM1A1) | (1 << WGM11);

TCCR1B = (1 << WGM12) | (1 << WGM13) |
         (1 << CS10); // Prescaler = 1

OCR1A = 512; // Set Duty Cycle to 50%
```
