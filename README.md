<img width="1892" height="1053" alt="palaOne" src="https://github.com/user-attachments/assets/0fdef5ba-eabd-4b71-9a0c-4c1dc78a4bee" />

# pala-one-firmware
Pala One — A tiny E-Ink reader project by Paul Lagier

The goal of the project was to create a simple, distraction-free reading device that feels minimal, portable and easy to build while still looking and behaving more like a real product than a typical DIY electronics project.

## Contributing

If you improve the firmware, add features or fix bugs, feel free to open a pull request.
Please clearly mention:
- which board version(s) you tested on (V1.1, V1.2, or both)
- what was changed
- how it was tested

## Board Versions

There are currently two supported Heltec Wireless Paper versions:
- `Heltec V1.1`
- `Heltec V1.2`

The board version is usually printed on the back of the PCB.

Both versions are built from the same source file (`Pala_One_2_1/Pala_One_2_1.ino`). Open it in Arduino IDE and uncomment the `#define` at the top that matches your hardware before compiling.

## Apps

Apps are compiled separately and uploaded to the device via the WiFi web interface — no firmware rebuild needed. The Apps menu discovers all `.bin` files in `/apps/` on the device and lists them by name.

### Building an app

You need:
- The `xtensa-esp32s3-elf-gcc` cross-compiler, which ships with the Arduino ESP32 board package. On Linux it is found under `~/.arduino15/packages/esp32/tools/esp-x32/<version>/bin/`; the `Makefile` locates it automatically.
- `python3` (for the post-build step that patches the entry point offset into the binary).

An app is a single C file that includes `pala_app.h` and `pala_api.h` from the firmware source and exports a `void app_main(const PalaAPI* api)` entry point. The header struct at the start of the binary carries the display name and API version check.

See `examples/click_counter/` for a complete working example with a `Makefile`.

```bash
cd examples/click_counter
make
# produces click_counter.bin
```

### Uploading an app

1. Select **Upload** from the library menu on the device.
2. Connect to the `PALA-XXXXXX` WiFi network (password: `palaread`).
3. Open `http://192.168.4.1` in a browser.
4. Use the **Upload app (.bin)** card to upload your `.bin` file.
5. Triple-click to exit upload mode — the app will appear in the Apps menu immediately.

### App API

Apps communicate with the firmware through the `PalaAPI` function pointer table passed to `app_main`. The current API version is **v3** (`PALA_API_VERSION 3` in `pala_app.h`).

#### Display

| Function | Description |
|---|---|
| `clearScreen()` | Clear the display buffer and prepare a new frame |
| `drawHeader(title)` | Draw the standard section header bar |
| `drawTextAt(x, y, text, bold)` | Draw text at a pixel position |
| `drawCenteredLarge(text)` | Draw text centred on screen in a large font |
| `refreshDisplay()` | Push the frame buffer to the e-ink panel |

#### Input

| Function | Description |
|---|---|
| `waitForEvent()` | Block until a button gesture; returns `PALA_CLICK` / `PALA_DOUBLE` / `PALA_TRIPLE` / `PALA_LONG` |
| `pollEvent()` | Non-blocking variant; returns 0 if no event is ready |
| `buttonPressed()` | Returns 1 if the button is currently held, 0 otherwise |
| `pendingPresses()` | Count of individual short press-release events since last call; bypasses multi-click grouping |

#### Timing

| Function | Description |
|---|---|
| `millisNow()` | Current uptime in milliseconds |
| `delayMs(ms)` | Yield for `ms` milliseconds |
| `rtcSeconds()` | Monotonic seconds counter that survives deep sleep; use for cross-session timing |

#### Storage

| Function | Description |
|---|---|
| `storageRead(key, buf, maxlen)` | Read from `/apps/{key}.dat`; returns bytes read, -1 on error |
| `storageWrite(key, buf, len)` | Write to `/apps/{key}.dat`; returns bytes written, -1 on error |

#### Utilities

| Function | Description |
|---|---|
| `snprintf_wrap(buf, len, fmt, ...)` | Standard `snprintf` |

Return from `app_main` to exit back to the Apps menu. Apps decide their own exit gesture — the firmware does not impose one.

**Constraints:**
- Apps must be compiled `-fPIC -mlongcalls` (position-independent).
- Apps must not use static mutable variables — the loader does not patch `.data` relocations.
- Maximum binary size: 48 KB.
- The `api_version` field in `PalaAppHeader` must match `PALA_API_VERSION` exactly — the firmware rejects mismatched binaries.

## Features

- TXT book support
- Adjustable font size
- Adjustable line spacing
- Deep sleep mode
- Reading progress saving
- USB-C charging
- Lightweight portable design
- Open-source firmware

# pala-one-japanese

Pala One — Japanese Edition, a modified version of [Paul Lagier's Pala One firmware](https://paullagier.craft.me/ereader2389232) for the Heltec Wireless Paper V1.2.

The goal of this fork is to extend the original firmware with Japanese language support and a flashcard mode for vocabulary study, while keeping the same minimal, distraction-free reading experience.

## Changes from Original

### Japanese Language Support
- Font replaced with `u8g2_font_b16_t_japanese3`, covering Hiragana, Katakana and a broad range of Kanji
- `TOP_PAD` increased from 0 to 4px to prevent the top of the first line being clipped on every page
- Selection highlight changed from bold text to an `*` prefix, as the new font does not have a bold variant

### Flashcard Mode
- Automatic detection — if a file contains the `===` delimiter the device enters flashcard mode with no configuration needed
- Long press jumps to a random card in the deck, always landing on the front (Japanese) side
- File scan on open is limited to the first 2KB so non-flashcard books open at normal speed
- `randomSeed` initialised on boot using the ESP32 microsecond timer for genuine randomness

## Flashcard File Format

```
猫
ねこ
---
Cat
猫が好きです - I like cats
===
山
やま
---
Mountain
あの山は高いです - That mountain is tall
===
```

- `---` separates the front (Japanese) side from the back (English) side
- `===` separates one card from the next
- Both delimiters act as invisible page breaks and do not appear on screen

### Navigation in Flashcard Mode

| Action | Result |
|---|---|
| Single press | Advance to next side or next card |
| Double press | Go back one side |
| Long press (850ms) | Jump to a random card, landing on the front side |

## Flashcard Decks

One JLPT study decks are included:

- `N5_kanji.txt` — 100 cards covering all JLPT N5 kanji, with readings and example sentences

## Board Version

This firmware supports the **Heltec Wireless Paper V1.2** only.

The board version is printed on the back of the PCB. For V1.1 support refer to the original Pala One firmware.

## Dependencies

Install the following in Arduino IDE via **Sketch → Include Library → Manage Libraries**:

| Library | Author |
|---|---|
| U8g2 | oliver |
| U8g2_for_Adafruit_GFX | oliver |
| heltec-eink-modules | Todd Herbert |
| Adafruit GFX Library | Adafruit |

## Reading Japanese Books

Upload plain `.txt` files saved as **UTF-8** via the device's built-in WiFi upload interface. Good sources for beginner Japanese content:

- **Tadoku Free Graded Readers** — [tadoku.org/japanese/free-books](https://tadoku.org/japanese/free-books/) — Level 0 books are almost entirely kana and completely free
- **NHK Web Easy** — [www3.nhk.or.jp/news/easy](https://www3.nhk.or.jp/news/easy/) — Real news in simple Japanese with furigana
- **Aozora Bunko** — [aozora.gr.jp](https://www.aozora.gr.jp) — Free public domain Japanese literature

## Features

- Japanese, Hiragana, Katakana and Kanji rendering
- Flashcard mode with random card selection
- TXT book support
- Adjustable font size
- Adjustable line spacing
- Deep sleep mode
- Reading progress saving
- USB-C charging

## Credits

- Original Pala One firmware by **Paul Lagier** — [project page](https://paullagier.craft.me/ereader2389232)
- Font: `u8g2_font_b16_t_japanese3` from the [U8g2 library](https://github.com/olikraus/u8g2) by oliver

## License & Copyright

This fork is based on Paul Lagier's Pala One firmware and is shared with his permission for personal and educational use.

Please do not:
- reupload paid project files from the original project
- redistribute complete download packages
- resell the project files
- commercially redistribute modified versions of paid assets

The original design, branding, documentation and paid project assets remain copyright © Paul Lagier.

---

Fork by Psycarius — based on the original Pala One by Paul Lagier

## Hardware

Pala One is based on:
- Heltec Wireless Paper
- 3D printed housing
- LiPo battery

## License & Copyright

The firmware in this repository is provided for personal and educational use.

Please do not:
- reupload paid project files
- redistribute complete download packages
- resell the project files
- commercially redistribute modified versions of paid assets

The design, branding, documentation and paid project assets remain copyright © Paul Lagier.

