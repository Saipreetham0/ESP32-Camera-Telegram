# ESP32-CAM → Telegram

Control an **ESP32-CAM (AI-Thinker)** from anywhere with a **Telegram bot**: send `/photo` and get a picture back in chat. No port forwarding, no cloud account, no app — Telegram's API is the whole backend.

## Bot commands

| Command | Action |
|---|---|
| `/start` | show available commands |
| `/photo` | capture a photo and send it to the chat |
| `/flash` | toggle the onboard flash LED (GPIO 4) |

Only the chat ID in your `secrets.h` gets a response — anyone else is told "Unauthorized user".

## Hardware

- **AI-Thinker ESP32-CAM** board (OV2640 camera) — pin mapping is already in the sketch
- **USB-serial adapter** (FTDI, CP2102…) for flashing — the board has no USB port:

| FTDI | ESP32-CAM |
|---|---|
| 5V | 5V |
| GND | GND |
| TX | U0R (GPIO 3) |
| RX | U0T (GPIO 1) |
| — | **GPIO 0 → GND** while flashing, disconnect after |

Uses UXGA (1600×1200) when PSRAM is detected, SVGA otherwise.

## Setup

### 1. Create the Telegram bot

1. Message [@BotFather](https://t.me/botfather) → `/newbot` → copy the **bot token**
2. Message [@myidbot](https://t.me/myidbot) → `/getid` → copy your **chat ID**
3. Press **Start** on your new bot (bots can't message you first)

### 2. Credentials

```bash
cp include/secrets.h.example include/secrets.h
# edit include/secrets.h — WiFi, bot token, chat ID
```

`include/secrets.h` is gitignored — real credentials never get committed.

### 3. Build & flash

```bash
pio run -t upload    # GPIO 0 grounded, then press RST
pio device monitor   # remove GPIO 0 jumper, press RST again
```

The serial monitor prints the IP once WiFi connects; then just message your bot.

## How it works

- The loop polls Telegram every second with `bot.getUpdates()` — outbound HTTPS only, which is why **no port forwarding or static IP** is needed
- Photos are streamed straight from the camera frame buffer to `api.telegram.org` as multipart form data in 1 KB chunks, so full images never sit in RAM
- The first frame after a request is discarded (sensor needs one frame to adjust exposure)
- Brown-out detection is disabled at boot — the camera + WiFi power spike falsely triggers it on most boards; use a solid 5V/2A supply regardless

## Troubleshooting

- **`Camera init failed 0x20001`** — camera ribbon cable not seated, or wrong board model
- **Bot never replies** — you didn't press Start on the bot, or chat ID ≠ the one in `secrets.h`
- **Random reboots when taking photos** — weak power supply; don't power from the FTDI's 3V3

## License

MIT
