# ESPHome: M5Stack AtomS3R + Atomic Echo Pyramid voice satellite for Home Assistant

A working [ESPHome](https://esphome.io) config that turns an **M5Stack AtomS3R** with the **Atomic Echo Pyramid** base into a **Home Assistant Assist voice satellite**, with on-device wake word, speaker output, touch controls, RGB lighting and a 128x128 screen showing animations for each assistant state.

This started from M5Stack's example YAML, which compiled but gave a blank screen and no working voice features for me. The fixes are listed below in case you hit the same problems.

## Features

- Home Assistant voice assistant satellite (STT, intent, TTS through your Assist pipeline)
- On-device wake words: `okay nabu`, `hey jarvis`, `hey mycroft` (or stream the wake word from Home Assistant)
- Wake word sensitivity and engine location selectable from Home Assistant
- Media player entity, volume control, timer alarm sound
- Mute microphone switch
- Touch strip: swipe on the left side for volume, on the right side for RGB brightness
- Pyramid RGB strips and LCD backlight exposed as lights
- Code-drawn screen animations (no image files needed):

| State | Animation |
|---|---|
| Idle | Breathing blue dot |
| Listening | Cyan sound-pulse rings expanding from a core |
| Thinking | Orange comet of dots orbiting the center |
| Replying | Green equalizer bars |
| Error | Flashing red ring with an X |
| No Home Assistant / No Wi-Fi | Pulsing magenta / orange ring |
| Initializing | White spinner |
| Muted | Mic-off icon |

## Hardware

- M5Stack AtomS3R (SKU C126)
- M5Stack Atomic Echo Pyramid (the Atom Voice Pyramid base)

## Tested with

- ESPHome 2026.10.2
- Home Assistant 2026.8.2 (Container)

Other versions may work but are untested. This config relies on M5Stack's external components (`aw87559`, `si5351`, `lp5562`, `pyramidrgb`, `pyramidtouch`) from
[m5stack/esphome-yaml](https://github.com/m5stack/esphome-yaml), loaded from `main`, so their changes can affect it.

## Quick start

1. Copy `secrets.example.yaml` to `secrets.yaml` in your ESPHome config folder and fill it in. Generate an API key with `openssl rand -base64 32`.
2. Copy `echo-pyramid.yaml` into your ESPHome config folder.
3. Compile and flash. To enter download mode on the AtomS3R, hold the reset button for about 2 seconds until the green LED lights up.
4. Adopt the device in Home Assistant (Settings, Devices & services, ESPHome) and choose your Assist pipeline.

## Screen: check which AtomS3R revision you have

AtomS3R units made **from May 2026 on use an ST7735 screen**; earlier units use a GC9107
([M5Stack AtomS3R docs](https://docs.m5stack.com/en/core/AtomS3R), version change 2026.05.14). M5Stack's page does not say how to tell them apart.

The config in this repo is the **ST7735** version and is confirmed working on a newer unit. ESPHome 2026.10.2 has no `M5STACK-ATOMS3R-ST7735` preset yet (it is being added in [esphome/esphome#18673](https://github.com/esphome/esphome/pull/18673)), so the display block rebuilds it by hand using that PR's values.

**If you have an older GC9107 unit** (untested by me), replace the whole `display:` block header, from `model:` down to `rotation:`, with:

```yaml
    model: M5STACK-ATOMS3R-GC9107
    dc_pin: GPIO42
    reset_pin: GPIO48
    cs_pin: GPIO14
```

**Symptom of the wrong chip:** backlight on, screen completely black (or colored noise).

Other display tweaks:

- Image upside down: remove `rotation: 180°`.
- Wrong colors: toggle `invert_colors` or add `color_order: rgb`.
- Thin band at one edge: adjust `pad_height` between 0 and 3.
- Still black or noisy: try the fallback config in the YAML comments, taken from [m5stack/esphome-yaml#104](https://github.com/m5stack/esphome-yaml/issues/104).

## What was changed from M5Stack's example

- Removed the `switch: platform: aw87559` block (`Platform not found: 'switch.aw87559'`). The top-level `aw87559:` hub entry stays.
- Trimmed the `si5351:` block to `id`, `i2c_id` and `address` (extra options caused an indentation/validation error with the current component).
- Set the media player pipeline to 16 kHz so it matches the shared mic/speaker I2S bus (the example asked for 48 kHz).
- Microphone set to a single channel (`left`). If the mic is silent, try `right`.
- Removed the unused `stop` and "hey peppa pig" wake word models.
- Fixed a compile error on newer ESPHome by casting the sound file pointer to `audio::AudioFile*` where `play_sound` is called.
- Replaced the screen images with code-drawn animations and updated the display config for the ST7735 panel.
- Moved the API key, OTA password and fallback hotspot password into `secrets.yaml`.

## Troubleshooting

| Error or symptom | Fix |
|---|---|
| `Platform not found: 'switch.aw87559'` | Remove the `switch: - platform: aw87559` block |
| Indentation or option error under `si5351:` | Keep only `id`, `i2c_id`, `address` |
| `invalid conversion from 'const esphome::audio::AudioFile*' to 'esphome::audio::AudioFile*'` | Use `const_cast<audio::AudioFile*>(id(...))` where `play_sound` is called, as in this config |
| `Unknown value 'M5STACK-ATOMS3R-ST7735'` | Your ESPHome predates the preset. Use the hand-built display block in this repo |
| Backlight on, screen black | Wrong screen chip. See the screen section above |
| Connects to Home Assistant but no voice features | Check the I2S pins and 16 kHz sample rates match this config, and look for `es7210`/`es8311` errors in the log |

## Credits

Based on M5Stack's example configuration and external components in
[m5stack/esphome-yaml](https://github.com/m5stack/esphome-yaml). Display values come from
[esphome/esphome#18673](https://github.com/esphome/esphome/pull/18673). Voice assistant structure follows
ESPHome's official voice satellite examples.

Written with help from Claude (Anthropic).

## License

[MIT](LICENSE). Portions are derived from M5Stack's
[esphome-yaml](https://github.com/m5stack/esphome-yaml) (also MIT, copyright m5stack), whose
notice is kept in the license file.
