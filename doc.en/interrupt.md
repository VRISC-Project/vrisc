# Interrupt

The conditions to trigger interrupt:

* machine exception
* external device

After an interrupt occured, CPU stores ip to x0, flags to x1,
then stick into kernel mode and disable interrupt.

* Interrupt codes:

int code|explanation
:-:|:-
1   |not effective address[IR_NOT_EFFECTIVE_ADDRESS]
2   |device[IR_DEVICES]
3   |clock[IR_CLOCK]
4   |unrecognized instruction[IR_INSTRUCTION_NOT_RECOGNIZED]
5   |permission denied[IR_PERMISSION_DENIED]
6   |io port invalid[IR_IO_PORT_INVALID]
7~32|reserved

## interrupt vector table (ivt)

Ivt consists of some address, that points to interrupt handlers.

Totally 32 entries.

Recommanded starting address of ivt is 0x0.

The first entry is magic number 0x0000_0000_0100_000a (not an effective entry).
