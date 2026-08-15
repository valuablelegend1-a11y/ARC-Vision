# ARC-Vision Build Journal

## 2026-08-15 — Design phase + repo setup

- Created the ARC-Vision repo and this first commit. Learning git properly this time instead of uploading files manually.
- Settled the core architecture: the glasses are a thin client. Audio/camera/sensor data streams over Wi-Fi to the desktop Jarvis (nanoGPT + Google STT + edge-tts), and TTS audio streams back to bone conduction. The ESP32-S3 cannot run the Jarvis brain, and doesn't need to.
- Picked the Seeed Studio XIAO ESP32-S3 Sense as the board:
  - 240 MHz dual-core ESP32-S3R8, 8 MB PSRAM (required for camera framebuffers), 8 MB flash
  - OV2640/OV3660 camera pre-wired on a flex connector (solves the camera pin-contention problem)
  - Onboard PDM microphone, microSD, U.FL antenna, onboard 1S LiPo charger
- Power strategy: deep sleep whenever idle (~3 mA floor with the Sense expansion attached). Wake on sound via an external electret + comparator → GPIO wake. Bone conduction amp gated on only while speaking; camera only initialized when a tool needs it; heart-rate sensor power-gated and sampled on demand.
- Known gotcha found during research: the Sense board ties the camera's RESET/PWDN pins to 3V3, so it cannot be power-gated by GPIO. Must call esp_camera_deinit() before deep sleep or the board can get stuck around 90 mA. Might add a physical transistor switch on the camera flex's 3V3 later.
- Emotion mapping idea confirmed: sample heart rate (MAX30102) in short bursts and combine with sentiment of my speech -> stressed (high HR + negative words) vs excited (high HR + positive words) vs calm.

## Next up:

- Pin map for the XIAO GPIOs (I2S out to MAX98357A, I2C for MAX30102, wake pin, battery ADC).
- Power budget worksheet with real part numbers.
- Then order parts and bench the audio path: mic -> Wi-Fi -> PC -> TTS -> bone conduction.
