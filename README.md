# NeoPixels-Shan
The progress of my coding and creating a code for Neopixels
import time
import random
from rpi_ws281x import Adafruit_NeoPixel, Color

# --------------------------------------------------
# NeoPixel LED Strip Configuration
# --------------------------------------------------
LED_COUNT = 300         # Number of NeoPixels on the strip
LED_PIN = 18            # GPIO pin connected to the pixels (PWM required)
LED_FREQ_HZ = 800000    # LED signal frequency (usually 800kHz)
LED_DMA = 10            # DMA channel for generating signal (use 10)
LED_BRIGHTNESS = 255    # Global brightness (0-255)
LED_INVERT = False      # True only if using inverting transistor

# Create NeoPixel object and initialize
strip = Adafruit_NeoPixel(LED_COUNT, LED_PIN, LED_FREQ_HZ,
                          LED_DMA, LED_INVERT, LED_BRIGHTNESS)
strip.begin()

# --------------------------------------------------
# Utility: Turn off all LEDs
# --------------------------------------------------
def turn_off_leds():
    """Turn off all LEDs by setting them to black (0,0,0)."""
    for i in range(strip.numPixels()):
        strip.setPixelColor(i, Color(0, 0, 0))
    strip.show()

# --------------------------------------------------
# Effect 1: Soft white pulse like jungle mist
# --------------------------------------------------
def soft_white_pulse(duration=5):
    """A soft white breathing effect to mimic a glowing jungle mist."""
    start_time = time.time()
    while time.time() - start_time < duration:
        # Gradually increase and decrease brightness
        for brightness in list(range(0, 256, 5)) + list(range(255, -1, -5)):
            color = Color(brightness, brightness, brightness)
            for i in range(strip.numPixels()):
                strip.setPixelColor(i, color)
            strip.show()
            time.sleep(0.02)

# --------------------------------------------------
# Effect 2: Rainbow wipe mimicking tropical colors
# --------------------------------------------------
def rainbow_wipe(duration=5):
    """A flowing rainbow wipe across the strip, like exotic jungle colors."""
    def wheel(pos):
        """Generate color from position (0-255)."""
        if pos < 85:
            return Color(pos * 3, 255 - pos * 3, 0)
        elif pos < 170:
            pos -= 85
            return Color(255 - pos * 3, 0, pos * 3)
        else:
            pos -= 170
            return Color(0, pos * 3, 255 - pos * 3)

    start_time = time.time()
    while time.time() - start_time < duration:
        for j in range(256):
            for i in range(strip.numPixels()):
                strip.setPixelColor(i, wheel((i + j) & 255))
            strip.show()
            time.sleep(0.02)

# --------------------------------------------------
# Effect 3: Comet tail - mimics fireflies or glowing bugs
# --------------------------------------------------
def comet_tail(duration=5):
    """A comet-like tail that bounces along the strip."""
    tail_length = 10
    start_time = time.time()
    position = 0
    direction = 1
    while time.time() - start_time < duration:
        for i in range(strip.numPixels()):
            # Brightness fades with distance from center
            brightness = max(0, 255 - abs(i - position) * (255 // tail_length))
            strip.setPixelColor(i, Color(brightness, brightness, brightness))
        strip.show()
        position += direction
        # Bounce back at edges
        if position >= strip.numPixels() or position < 0:
            direction *= -1
            position += direction
        time.sleep(0.02)

# --------------------------------------------------
# Effect 4: Sparkle - random twinkling fireflies
# --------------------------------------------------
def sparkle_effect(duration=5):
    """Random white sparkles like twinkling fireflies."""
    start_time = time.time()
    while time.time() - start_time < duration:
        i = random.randint(0, strip.numPixels() - 1)
        strip.setPixelColor(i, Color(255, 255, 255))
        strip.show()
        time.sleep(0.05)
        strip.setPixelColor(i, Color(0, 0, 0))
        strip.show()
        time.sleep(0.05)

# --------------------------------------------------
# Effect 5: Color chase - jungle creatures dashing past
# --------------------------------------------------
def color_chase(duration=5):
    """A fast red dot running across the strip like jungle motion."""
    start_time = time.time()
    color = Color(255, 0, 0)  # Bright red
    while time.time() - start_time < duration:
        for i in range(strip.numPixels()):
            strip.setPixelColor(i, color)
            if i >= 1:
                strip.setPixelColor(i - 1, Color(0, 0, 0))
            strip.show()
            time.sleep(0.01)

# --------------------------------------------------
# Effect 6: Fade to black - smooth wind-down ending
# --------------------------------------------------
def fade_to_black(duration=5):
    """Gradually fade all LEDs to black for a clean ending."""
    for brightness in range(255, -1, -5):
        color = Color(brightness, brightness, brightness)
        for i in range(strip.numPixels()):
            strip.setPixelColor(i, color)
        strip.show()
        time.sleep(duration / (255 / 5))

# --------------------------------------------------
# Main Sequence Runner
# --------------------------------------------------
def run_sequence():
    """Run a full 30-second jungle-themed lighting sequence."""
    try:
        soft_white_pulse(duration=5)
        rainbow_wipe(duration=5)
        comet_tail(duration=5)
        sparkle_effect(duration=5)
        color_chase(duration=5)
        fade_to_black(duration=5)
    except KeyboardInterrupt:
        # Gracefully turn off LEDs if interrupted
        turn_off_leds()

# --------------------------------------------------
# Entry Point
# --------------------------------------------------
if __name__ == "__main__":
    run_sequence()
    turn_off_leds()
