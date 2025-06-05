Shan's Neopixel Sequence EGL 313

Temple Awakening Light Show
This project is a custom NeoPixel light show designed to run on a Raspberry Pi using the rpi_ws281x library. It controls 300 addressable LEDs wrapped around a pedestal, syncing dramatic lighting effects to a soundtrack. The current version includes a complete 40-second sequence with four scenes: a soft blue-brown pulse, an inward red-orange ripple, a glowing earth shimmer with white sparks, and a bright white-to-gold finale. Each scene represents a step in an ancient temple's awakening. The code is optimized for live performances and synced visual storytelling.

Here's the code for the sequence: 
import time
from rpi_ws281x import Adafruit_NeoPixel, Color

# --- LED strip configuration ---
LED_COUNT = 300
LED_PIN = 18
LED_FREQ_HZ = 800000
LED_DMA = 10
LED_BRIGHTNESS = 255
LED_INVERT = False

strip = Adafruit_NeoPixel(LED_COUNT, LED_PIN, LED_FREQ_HZ, LED_DMA, LED_INVERT, LED_BRIGHTNESS)
strip.begin()

# --- Color Helper ---
def color(r, g, b):
    return Color(r, g, b)

def turn_off_leds():
    for i in range(strip.numPixels()):
        strip.setPixelColor(i, Color(0, 0, 0))
    strip.show()

# --- Colors (Temple Themed) ---
DEEP_BLUE = color(0, 105, 148)
BURNT_AMBER = color(139, 69, 19)
CRIMSON = color(178, 34, 34)
PALE_GOLD = color(255, 236, 179)
SOFT_WHITE = color(240, 240, 240)

# --- Scene 1: Deep pulse (awakening energy) ---
def blue_brown_pulse(duration=10):
    start = time.time()
    while time.time() - start < duration:
        for b in list(range(30, 100, 5)) + list(range(100, 30, -5)):
            col = color(int(b * 0.0), int(b * 0.4), b)  # Deep bluish pulse
            for i in range(strip.numPixels()):
                if i % 3 == 0:
                    strip.setPixelColor(i, BURNT_AMBER)
                else:
                    strip.setPixelColor(i, col)
            strip.show()
            time.sleep(0.03)

# --- Scene 2: Ripple inward with red and amber ---
def fire_ripple_inward(duration=10):
    center = strip.numPixels() // 2
    start = time.time()
    while time.time() - start < duration:
        for offset in range(center):
            if offset % 2 == 0:
                col = CRIMSON
            else:
                col = BURNT_AMBER
            strip.setPixelColor(center + offset, col)
            strip.setPixelColor(center - offset, col)
            if offset > 0:
                strip.setPixelColor(center + offset - 1, Color(0, 0, 0))
                strip.setPixelColor(center - offset + 1, Color(0, 0, 0))
            strip.show()
            time.sleep(0.01)

# --- Scene 3: Earth shimmer with white sparks ---
def shimmer_with_sparks(duration=10):
    import random
    start = time.time()
    while time.time() - start < duration:
        for i in range(strip.numPixels()):
            if random.random() < 0.05:
                strip.setPixelColor(i, SOFT_WHITE)
            else:
                strip.setPixelColor(i, BURNT_AMBER)
        strip.show()
        time.sleep(0.1)

# --- Scene 4: Final white burst to gold (Divine Awakening) ---
def white_to_gold_burst(duration=10):
    steps = int(duration / 0.05)
    for i in range(steps):
        ratio = i / steps
        r = int(240 * (1 - ratio) + 255 * ratio)
        g = int(240 * (1 - ratio) + 236 * ratio)
        b = int(240 * (1 - ratio) + 179 * ratio)
        col = color(r, g, b)
        for j in range(strip.numPixels()):
            strip.setPixelColor(j, col)
        strip.show()
        time.sleep(0.05)

# --- Storyboard Sequence ---
def run_temple_awakening_sequence():
    try:
        blue_brown_pulse(duration=10)           # 1:00–1:10
        fire_ripple_inward(duration=10)         # 1:10–1:20
        shimmer_with_sparks(duration=10)        # 1:20–1:30
        white_to_gold_burst(duration=10)        # 1:30–1:40
    except KeyboardInterrupt:
        turn_off_leds()

# --- Run the Light Show ---
if __name__ == "__main__":
    run_temple_awakening_sequence()
    turn_off_leds()




What the Code Does?

This Python script runs a series of lighting effects on 300 NeoPixel LEDs connected to a Raspberry Pi. The lights are programmed to simulate a story where a temple awakens, using different lighting scenes and colors.

Setup:
strip = Adafruit_NeoPixel(...)
strip.begin()
This sets up the NeoPixel strip with 300 LEDs. It tells the Raspberry Pi how many LEDs there are, which pin they're connected to, how bright to make them, and then prepares the strip to receive commands.

Color Function: 
def color(r, g, b):
    return Color(r, g, b)
This is a helper function to make it easy to create colors using red, green, and blue values.

Defined Colors:
DEEP_BLUE = color(0, 105, 148)
BURNT_AMBER = color(139, 69, 19)
...
These are preset colors chosen to match the theme of an ancient temple. They're used in different scenes.

Scene 1: blue_brown_pulse()
This scene runs for 10 seconds. It gradually brightens and dims a bluish light while mixing in warm amber lights at regular positions. The effect looks like a gentle pulse, representing the temple beginning to wake up.

Scene 2: fire_ripple_inward()
This creates a ripple of red and amber colors starting from the center of the strip and moving outward. Each ripple flashes quickly, alternating colors, and then turns off. It simulates energy being pulled into the center, as if the temple is activating.

Scene 3: shimmer_with_sparks()
This scene simulates a flickering ground. Most LEDs glow in a soft amber tone, but random LEDs briefly flash white like sparks. The effect feels earthy and magical.

Scene 4: white_to_gold_burst()
This is the final scene. It starts with all the LEDs glowing white, then slowly transitions them to a warm gold. This represents a divine or powerful awakening moment at the end of the sequence.

Sequence Control
python
Copy
Edit
def run_temple_awakening_sequence():
This function runs all four scenes in order, each lasting 10 seconds. After it’s done, it turns off all the LEDs.

Program Entry Point
python
Copy
Edit
if __name__ == "__main__":
This makes sure the sequence starts only when the script is run directly. It plays the full light show and turns everything off at the end.


