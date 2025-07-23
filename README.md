# Raspberry Pi Media Display with GPIO Control

This project is a fullscreen media display application for Raspberry Pi that plays images and videos from customizable playlists. It supports physical button control via GPIO and can auto-switch playlists based on the time of day.

---

## Features

- Fullscreen display of images and videos using Pygame and VLC
- Smooth video playback using python-vlc (software-rendered)
- Three GPIO buttons for:
  - Image-only playlist
  - Video-only playlist
  - Mixed playlist (images + videos)
- Optional scheduler that changes the playlist based on time:
  - Morning (from 08:00)
  - Afternoon (from 13:00)
  - Evening (from 18:00)
- Auto-scaling and centered image display
- Auto-loop image mode with timing control

---

## Hardware Requirements

- Raspberry Pi 3, 4, or newer with GPIO and HDMI output
- HDMI-connected monitor or TV
- 3 physical momentary push buttons
- Breadboard and jumper wires (optional but useful for prototyping)
- Power supply for Raspberry Pi

**GPIO Button Assignments (BCM mode):**

| Button Function  | GPIO Pin |
|------------------|----------|
| Image Mode       | 17       |
| Video Mode       | 27       |
| Mixed Playlist   | 22       |

---

## Software Requirements

Install the necessary libraries using the following commands:

```bash
sudo apt update
sudo apt install python3-pygame python3-vlc
pip3 install RPi.GPIO
