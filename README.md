Shan's Neopixel Sequence EGL 314

Temple Awakening Light Show
This project is a custom NeoPixel light show designed to run on a Raspberry Pi using the rpi_ws281x library. It controls 300 addressable LEDs wrapped around a pedestal, syncing dramatic lighting effects to a soundtrack. The current version includes a complete 40-second sequence with 7 scenes. 


## Features
- 7 lighting scenes (fade-in, pulse, ripple, shimmer, chase, spiral, fade-out)
- Fully customizable color themes
- OSC message support to trigger actions on other devices

## Setup
1. Connect NeoPixel strip to GPIO18 with proper power supply
2. Install dependencies:
   ```bash
   pip install -r requirements.txt


Here's the code: 

import time
from rpi_ws281x import Adafruit_NeoPixel, Color
from pythonosc import udp_client
import random

# --- LED strip configuration ---
LED_COUNT = 300            # We have 300 tiny lights on the strip
LED_PIN = 18               # The strip is plugged into pin 18 on the Raspberry Pi
LED_FREQ_HZ = 800000       # How fast the lights talk to the Pi (speed)
LED_DMA = 10               # A technical thing to help lights work smoothly
LED_BRIGHTNESS = 255       # How bright the lights should be (max brightness)
LED_INVERT = False         # Don't flip the signal to the lights

# --- Customizable Color Theme --- 
# These are the special colors we will use
BASE_COLOR       = (0, 105, 148)     # Deep Blue color
ACCENT_COLOR     = (139, 69, 19)     # Burnt Amber color (like orange-brown)
SPARKLE_COLOR    = (240, 240, 240)   # Soft White sparkle color
FINAL_BURST_COLOR = (255, 236, 179)  # Pale Gold for the big finish
INTRO_COLOR      = (255, 0, 255)     # Bright Magenta (pinkish purple)

# --- OSC Setup --- (talking to another computer before the show)
def send_message(receiver_ip, receiver_port, address, message):
    try:
        client = udp_client.SimpleUDPClient(receiver_ip, receiver_port)  # Prepare to send message
        client.send_message(address, message)                            # Send the message
        print("Message sent successfully.")                             # Tell us it worked
    except:
        print("Message not sent")                                       # If no, say it failed

# This is the address of the other computer and the message we send it:
PI_A_ADDR = "192.168.254.40"
PORT = 8000
addr = "/action/40044"
msg = float(1)

# Send hello message to the other computer before we start the lights
send_message(PI_A_ADDR, PORT, addr, msg)

# --- LED Setup --- (getting the magic lights ready)
strip = Adafruit_NeoPixel(LED_COUNT, LED_PIN, LED_FREQ_HZ, LED_DMA, LED_INVERT, LED_BRIGHTNESS)
strip.begin()   # Turn on the magic wand and get it ready

# --- Helper Functions ---
def color(r, g, b):
    # Turn red, green, blue numbers into a color the lights understand
    return Color(r, g, b)

def turn_off_leds():
    # Turn off all the lights (make them black)
    for i in range(strip.numPixels()):
        strip.setPixelColor(i, Color(0, 0, 0))
    strip.show()

# --- Scene 0: Intro Fade from INTRO_COLOR → BASE_COLOR ---
def intro_fade_to_base(duration=6):
    steps = int(duration / 0.03)                 # How many little steps we take to change colors
    for i in range(steps):
        ratio = i / steps                         # A number going from 0 to 1 to help fade colors
        r = int((1 - ratio) * INTRO_COLOR[0] + ratio * BASE_COLOR[0])   # Mix red from pink to blue
        g = int((1 - ratio) * INTRO_COLOR[1] + ratio * BASE_COLOR[1])   # Mix green from pink to blue
        b = int((1 - ratio) * INTRO_COLOR[2] + ratio * BASE_COLOR[2])   # Mix blue from pink to blue
        col = color(r, g, b)                      # Make the new color
        for j in range(strip.numPixels()):       # For each light on the strip
            strip.setPixelColor(j, col)          # Set the light to the new color
        strip.show()                             # Show the new colors on the strip
        time.sleep(0.03)                         # Wait a tiny bit before next step

# --- Scene 1: Pulse Beat ---
def base_accent_pulse(duration=5.5):
    start = time.time()
    while time.time() - start < duration:
        # Make brightness go up and down smoothly
        for b in list(range(30, 100, 5)) + list(range(100, 30, -5)):
            # Make a blue-ish color with changing brightness
            col = color(int(b * 0.0), int(b * 0.4), b)  
            for i in range(strip.numPixels()):
                # Every third light shines amber, others pulse blue
                strip.setPixelColor(i, color(*ACCENT_COLOR) if i % 3 == 0 else col)
            strip.show()
            time.sleep(0.03)

# --- Scene 2: Ripple Inward ---
def ripple_inward(duration=5.5):
    center = strip.numPixels() // 2              # Find the middle light
    start = time.time()
    while time.time() - start < duration:
        for offset in range(center):
            # Alternate colors orange and blue on the ripple wave
            col = color(*BASE_COLOR) if offset % 2 == 0 else color(*ACCENT_COLOR)
            strip.setPixelColor(center + offset, col)  # Light on right side from middle
            strip.setPixelColor(center - offset, col)  # Light on left side from middle
            if offset > 0:
                # Turn off lights that were lit before (to make ripple move)
                strip.setPixelColor(center + offset - 1, Color(0, 0, 0))
                strip.setPixelColor(center - offset + 1, Color(0, 0, 0))
            strip.show()
            time.sleep(0.01)

# --- Scene 3: Sparkle Party ---
def shimmer_with_sparks(duration=5.5):
    start = time.time()
    while time.time() - start < duration:
        for i in range(strip.numPixels()):
            # Randomly sparkle some lights white, others amber
            strip.setPixelColor(i, color(*SPARKLE_COLOR) if random.random() < 0.05 else color(*ACCENT_COLOR))
        strip.show()
        time.sleep(0.1)

# --- Scene 4: Color Chase ---
def color_chase(duration=5.5):
    start = time.time()
    pos = 0
    while time.time() - start < duration:
        turn_off_leds()                 # First, turn all lights off
        # One blue light moves forward, one amber follows behind
        strip.setPixelColor(pos % LED_COUNT, color(0, 105, 255))      # Bright blue
        strip.setPixelColor((pos - 5) % LED_COUNT, color(255, 140, 0)) # Orange trailing
        strip.show()
        pos += 1                       # Move forward
        time.sleep(0.03)

# --- Scene 5: Spiral Pulse ---
def spiral_pulse(duration=5.5):
    center = strip.numPixels() // 2
    start = time.time()
    while time.time() - start < duration:
        for offset in range(center):
            turn_off_leds()
            # White lights flash moving outwards and back
            strip.setPixelColor(center + offset, Color(255, 255, 255))
            strip.setPixelColor(center - offset, Color(255, 255, 255))
            strip.show()
            time.sleep(0.01)

# --- Scene 6: Fade Out to Black ---
def final_fade_out(duration=5.5):
    steps = int(duration / 0.05)
    for i in range(steps):
        ratio = 1 - (i / steps)  # Goes from 1 down to 0 (bright to dark)
        r = int(BASE_COLOR[0] * ratio)
        g = int(BASE_COLOR[1] * ratio)
        b = int(BASE_COLOR[2] * ratio)
        col = color(r, g, b)      # Color gets darker each step
        for j in range(strip.numPixels()):
            strip.setPixelColor(j, col)
        strip.show()
        time.sleep(0.05)

# --- Master Sequence: run all Scenes one after another ---
def run_temple_sequence():
    try:
        base_accent_pulse(duration=5.5)   # Scene 1: pulsing lights
        ripple_inward(duration=5.5)       # Scene 2: ripple wave
        shimmer_with_sparks(duration=5.5) # Scene 3: sparkles
        color_chase(duration=5.5)         # Scene 4: running lights
        spiral_pulse(duration=5.5)        # Scene 5: swirling white pulse
       
        final_fade_out(duration=5.5)      # Scene 6: slow fade to black
    except KeyboardInterrupt:
        turn_off_leds()                   # If we stop the program, turn off lights nicely

# --- Execution ---
if __name__ == "__main__":
    try:
        intro_fade_to_base(duration=6)   # First, fade from magenta to blue
        run_temple_sequence()            # Then, play all the dance moves
    finally:
        turn_off_leds()                  # At the end, turn off all lights. 



