Notes:


Switch debouncing


## AVR Stack
* in data memory
* PUSH/POP and CALL/RET
* Stack Pointer is always pointing at first FREE entry
*  Decremented in PUSH instructions
> Note that the Stack is implemented as growing from higher memory locations to lower memory locations.
* low byte pushed first then high byte (e.g. 1235 -> 35 then 12 on top)
* always need to balance PUSH and POP
> The Stack Pointer SP is read/write accessible in the I/O space

### Stack Initialization and Allocation

Note: interrupts are disabled when creating the stack.

**Preferred method** (init + allocate) that **clearly defines the size** of the stack:
```
; initialization of the stack
.CSEG

LDI    R16, LOW (TopOfStack)
OUT    SPL, R16
LDI    R16, HIGH (TopOfStack)
OUT    SPH, R16

; allocation of the stack

.EQU    STACK_SIZE = 32
.DSEG

            .BYTE STACK_SIZE - 1
TopOfStack: .BYTE 1
```

Faster method with **no transparency** **on sizing**: 
```
.CSEG

LDI    R16, LOW (RAMEND)
OUT    SPL, R16
LDI    R16, HIGH (RAMEND)
OUT    SPH, R16
```

### Stack Frame 
* Uses a **Frame Pointer** to point to the elements in a procedure (e.g. local variables and arguments)
* in AVR the Y register is used as the frame pointer
* Values are **NEVER erased in the stack**, just overwritten. When popping or pushing values, only the pointer is modified
* Memory is dynamically used

## Timers
Use cases:
* Timed interrupt
* Pulse generator for synchronization or sampling
* Sound
* PWM
### Design
#### Counter
* Because of prescaler, don't want to count ever clock
* CompEq signal used to determine to reset counter
* PWM takes two compare registers, square wave takes 1, also used for comparator

#### Prescaler
* Divides the clock frequency (e.g. every 2 clocks or every 8 clocks, count instead)
* Has select option for count enable
* Use to reduce necessity of needing high bit space for low frequencies (e.g. 1.2Hz = 24 bit counter)
## Interrupt
* **IRQ**: maskable interrupt, normally used
* **NMI**: non-maskable interrupt, edge sensitive, catastrophic events
Checking interrupts:
* **fixed address**: each interrupt has an address and the CPU executes at this address when given the interrupt
	* only couple of words in memory, so it is just a jump to the actual event handler. often in ROM
* **Vector table** : each entry is an address of an event handler. often in RAM

### CPU Flow:

| Typical CPU Operation      | AVR Equivalent                                  |     |     |
| -------------------------- | ----------------------------------------------- | --- | --- |
| Finish current instruciton | Finish instruction                              |     |     |
| Save processor state       | Push PC                                         |     |     |
| Disable further interrupts | Clear interrupt flag                            |     |     |
| Acknowledge interrupt      |                                                 |     |     |
| Invoke Event Handler       | Jump to event handler in interrupt vector table |     |     |
See: AVR Interrupt Vector Table and AVR Interrupt Response Time


## Event Handlers 
| Typical CPU Operation                            | AVR Equivalent               |     |
| ------------------------------------------------ | ---------------------------- | --- |
| Save program state                               | Push all used registers      |     |
| Process event                                    | Process event                |     |
| Acknowledge interrupt                            |                              |     |
| Restore program state                            | Pop all pushed registers     |     |
| Restore processor state and re-enable interrupts | Return from interrupt (RETI) |     |
* By default event handlers disable interrupts. In the AVR manual, it states
>The user software can write logic one to the I-bit to enable nested interrupts
* Use **shared variables** to communicate between interrupts and main code
	* Critical code when shared variables are accessed by foreground code and interrupts
* Event handlers usually **only SET FLAGs**. Foreground code resets flags.
* To protect critical code from interrupts:
```
IN     R0, SREG       ; save interrupt flag status
CLI                   ; prevent interrupts
; critical code here
OUT    SREG, R0       ; restore interrupt flag status
``` 
* DON'T do the following because we never know if interrupts are actually enabled. We just want to disable them and restore to whatever previous state it was before critical code was executed:
```
CLI                   ; prevent interrupts
; critical code here
SEI                   ; re-enable interrupts
```

## Parallel I/O
Uses:
* switches
* keypads
* LEDs
* LCD
* Motor
* DAC
### Design
* Not sure the number of input/outputs, so make the port programmable in one direction
*  Limited number of pins, so make the pins multiple uses (e.g. parallel I/O and SPI)
### Hardware
* Bit or byte control on the pin
* Latched output (saves)
* input reads either pin or output register


## Switches

### Hardware

### Software

### Debouncing
* Switches bounce when pressed (think of noise), so need to measure how long the switch is actually down for the correct signal
* Note press down is 0
#### Hardware Debouncing
#### Software Debouncing
```
IF (switch up) THEN
	debounce_counter = DEBOUNCE_TIME
ELSE 
	debounce_counter--
	IF (debounce_counter == 0) THEN
		set switchFlag
	ELSE IF (debounce_counter < 0) THEN
		debounce_counter = 0
	
```
We can add **AUTO REPEAT**, which is essentially if the switch is held down, multiple presses are registered with a **REPEAT_RATE** duration:
```
IF (switch up) THEN
	debounce_counter = DEBOUNCE_TIME
ELSE 
	debounce_counter--
	IF (debounce_counter == 0) THEN
		set switchFlag
		debounce_counter = REPEAT_RATE
```
We can add variable rate auto-repeat
```
IF (switch up) THEN
	debounce_counter = DEBOUNCE_TIME
	repeat_rate = SLOW rate
	repeat_counter = FAST_REPEAT_TIME
ELSE 
	debounce_counter--
	IF (debounce_counter == 0) THEN
		set switchFlag
		debounce_counter = REPEAT_RATE
	decrement repeat_counter
	IF (repeat_counter = 0) THEN
		repeat_rate = FAST_RATE
```
Think of this when you hold down the space bar key on a word editor...the first few time intervals the space moves slowly (repeat slow rate) then very quickly (repeat fast rate)