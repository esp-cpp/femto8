# ESP32 port

The glue that runs femto8 on the esp-cpp emulators: `pico8.cpp` runs the cart
in its own task, presents its frames, feeds its audio and buttons, pauses it
for the emulator menu and resets femto8's statics between launches;
`platform/` holds the seams femto8's sources include (`p8_espbox.h`, the Lua
allocator, the save-state hooks). It is shared by:

- [esp-box-emu](https://github.com/esp-cpp/esp-box-emu) (ESP32-S3)
- [tab5-emu](https://github.com/esp-cpp/tab5-emu) (ESP32-P4)

Each project builds `src/` and `port/` as its `pico8` component, with
`FEMTO8_ESPBOX=1`, `CARTDATA_PATH` and `DEFAULT_CARTS_PATH`, and a linker
fragment that places femto8's statics and brackets them with the
`_pico8_bss/_common/_data` SURROUND symbols `pico8.cpp` resets. On RISC-V
(P4) build with `-msmall-data-limit=0` so no static lands in `.sbss`/`.sdata`
outside those bounds. The project supplies `box-emu.hpp` (a `BoxEmu`
singleton: `push_frame`, `frame_slot_free`, `frame_buffer0/1`, `palette`,
`gamepad_state_latched`, `audio_sample_rate`, `play_audio`, `Bsp`).

## Compile-time options

| Define | Default | Effect |
|---|---|---|
| `PICO8_AUDIO_VIA_EMU` | 1 on the P4, else 0 | Send audio through `BoxEmu::play_audio()` (which resamples to a fixed-rate DAC) instead of straight into the BSP's stream at 44.1 kHz |
| `PICO8_PROFILE` | off | Lua instructions/s, GC share and per-API cycle counts with the fps line |
