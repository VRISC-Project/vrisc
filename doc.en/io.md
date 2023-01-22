# io

IO port is the main way to access external device.

In CPU, it use `in` and `out` instruction to access port `0` to `63` (totally 64 ports).

## Stable ports

port|using|explanation
:-:|:-|:-
0   |Post interrupt        |Read this port after IR_DEVICES occured.
1   |Multi-processor wakeup |Write 1-byte integer `n`, and `core#n` will start.
