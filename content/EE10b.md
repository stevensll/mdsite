Notes:


Switch debouncing


## AVR Stack
* in data memory
* PUSH/POP and CALL/RET
* Stack Pointer is always pointing at first FREE entry
*  Decremented in PUSH instructions  - "Note that the Stack is implemented as growing from higher memory loca tions to lower memory locations."
* low byte pushed first then high byte (e.g. 1235 -> 35 then 12 on top)
* always need to balance PUSH and POP
* "The Stack Pointer SP is read/write accessible in the I/O space" in AVR

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


## Interrupt
* **IRQ**: maskable interrupt, normally used
* **NMI**: non-maskable interrupt, edge sensitive, catastrophic events
Checking interrupts:
* **fixed address**: each interrupt has an address and the CPU executes at this address when given the interrupt
	* only couple of words in memory, so it is just a jump to the actual event handler. often in ROM
* **Vector table** : each entry is an address of an event handler. often in RAM

### CPU Flow:
1) Finish current instruction (typically 1 clock)
2) Save processor state
3) Disable further interrupts
4) Acknowledge interrupt
5) Invoke event handler
See: AVR Interrupt Vector Table
