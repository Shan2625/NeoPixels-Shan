FUll CODE WITH COMMENTS :



import pygame
import os
import time
import RPi.GPIO as GPIO
import threading
import vlc

# === CONFIGURATION ===
ENABLE_SCHEDULER = False #Scheduler on/off

# === GPIO setup ===
IMAGE_BUTTON = 17
VIDEO_BUTTON = 27
PLAYLIST_BUTTON = 22
GPIO.setmode(GPIO.BCM) #GPIO.BCM (Functional Numbering)
GPIO.setup(IMAGE_BUTTON, GPIO.IN, pull_up_down=GPIO.PUD_UP)  #The pin is always HIGH until you press a button, which connects it to Ground and forces it LOW
GPIO.setup(VIDEO_BUTTON, GPIO.IN, pull_up_down=GPIO.PUD_UP)
GPIO.setup(PLAYLIST_BUTTON, GPIO.IN, pull_up_down=GPIO.PUD_UP)

# === Globals ===
screen_mode = "landscape"
paused = False # Note: Pause functionality is disabled in this version.
playlist_running = False
restart_playlist = False
playlist_thread = None

# Playlist paths
PLAYLIST_FILE = "/home/pi/media_display_project/Playlist"
IMAGE_PLAYLIST_FILE = "/home/pi/media_display_project/Images"
VIDEO_PLAYLIST_FILE = "/home/pi/media_display_project/Video"

# Scheduler playlist files
MORNING_FILE = '/home/pi/media_display_project/Morning'
AFTERNOON_FILE = '/home/pi/media_display_project/Afternoon'
EVENING_FILE = '/home/pi/media_display_project/Evening'
MORNING_TIME = '08:00'
AFTERNOON_TIME = '13:00'
EVENING_TIME = '18:00'



pygame.init()
info = pygame.display.Info()
screen_width, screen_height = info.current_w, info.current_h
screen = pygame.display.set_mode((screen_width, screen_height), pygame.FULLSCREEN)
pygame.mouse.set_visible(False)
clock = pygame.time.Clock()
font = pygame.font.SysFont('Arial', 28)

# Auto-loop image control
auto_loop = False
image_index = 0
last_switch_time = time.time()

def get_time_based_playlist():
    current_time = time.strftime('%H:%M')
    if current_time >= EVENING_TIME: return EVENING_FILE
    elif current_time >= AFTERNOON_TIME: return AFTERNOON_FILE
    elif current_time >= MORNING_TIME: return MORNING_FILE
    else: return MORNING_FILE

def load_playlist(path, expected_type=None):
    items = []
    if not os.path.exists(path):
        print(f"[ERROR] Playlist file not found: {path}")
        return items
    with open(path, 'r') as file:
        for line in file:
            parts = line.strip().split(',')
            if len(parts) >= 2:
                media_type, file_path = parts[0].lower(), parts[1].strip()
                duration = float(parts[2]) if media_type == 'image' and len(parts) > 2 else 3
                if expected_type and media_type != expected_type: continue
                items.append({'type': media_type, 'file': file_path, 'duration': duration})
    return items

def display_image(path): #This function safely loads an image, resizes it to fit perfectly in the center of the screen without stretching, and then updates the display to show it.
    screen.fill((0, 0, 0))
    try:
        image = pygame.image.load(path).convert()
        if screen_mode == 'portrait': image = pygame.transform.rotate(image, 90)
        img_rect = image.get_rect()
        screen_rect = screen.get_rect()
        scale = min(screen_rect.width / img_rect.width, screen_rect.height / img_rect.height)
        new_size = (int(img_rect.width * scale), int(img_rect.height * scale))
        image = pygame.transform.smoothscale(image, new_size)
        img_rect = image.get_rect(center=screen_rect.center)
        screen.blit(image, img_rect)
        pygame.display.flip()
    except Exception as e:
        print(f"[ERROR] Displaying image: {e}")

def stop_video():
    pass # The new play_single_video handles its own cleanup.

def check_events():
    global playlist_running
    for event in pygame.event.get():
        if event.type == pygame.KEYDOWN and event.key == pygame.K_ESCAPE:
            playlist_running = False # Signal thread to stop
            return False # Signal main loop to exit
    return True # Signal main loop to continue

def play_single_video(video_path, window_id):
    """
    Creates a new VLC instance, plays one video, and then destroys the instance.
    This is the "kill and restart" method.
    """
    vlc_instance = vlc.Instance('--vout=x11 --no-video-title-show')
    vlc_media_player = vlc_instance.media_player_new()
    vlc_media_player.set_xwindow(window_id)

    screen.fill((0, 0, 0))
    pygame.display.flip()

    media = vlc_instance.media_new(video_path)
    vlc_media_player.set_media(media)
    vlc_media_player.play()
    vlc_media_player.set_fullscreen(True)
    time.sleep(0.5)

    playing = True
    while playing and playlist_running:
        state = vlc_media_player.get_state()
        if state in [vlc.State.Ended, vlc.State.Stopped, vlc.State.Error]:
            playing = False
        time.sleep(0.1)

    if vlc_media_player.is_playing():
        vlc_media_player.stop()
    vlc_media_player.release()
    vlc_instance.release()
    
    screen.fill((0, 0, 0))
    pygame.display.flip()


def playlist_loop(items):
    global playlist_running, restart_playlist
    playlist_running = True
    idx = 0
    
    window_id = pygame.display.get_wm_info()['window']

    if not items:
        print("[WARNING] Playlist is empty. Thread exiting.")
        return

    while playlist_running:
        if restart_playlist:
            print("[PLAYLIST] Restart triggered.")
            idx = 0
            restart_playlist = False

        if idx >= len(items):
            idx = 0
            print("[INFO] Playlist finished. Looping...")
            continue

        item = items[idx]
        if not os.path.exists(item['file']):
            print(f"[MISSING] {item['file']}")
        elif item['type'] == 'image':
            display_image(item['file'])
            start = time.time()
            while time.time() - start < item['duration']:
                if restart_playlist or not playlist_running: break
                time.sleep(0.05)
        elif item['type'] == 'video':
            play_single_video(item['file'], window_id)

        idx += 1


def main():
    global playlist_running, restart_playlist, playlist_thread, auto_loop, image_index, last_switch_time

    # Pre-load the image list so we don't have to do it repeatedly
    image_items = load_playlist(IMAGE_PLAYLIST_FILE, 'image')

    if ENABLE_SCHEDULER:
        items = load_playlist(get_time_based_playlist())
        if items:
            print(f"[SCHEDULER] Auto-playing: {get_time_based_playlist()}")
            playlist_thread = threading.Thread(target=playlist_loop, args=(items,))
            playlist_thread.start()

    running = True
    while running:
        running = check_events() # Handle exit button

        if GPIO.input(IMAGE_BUTTON) == GPIO.LOW:
            print("[BUTTON] IMAGE pressed")
            while GPIO.input(IMAGE_BUTTON) == GPIO.LOW: time.sleep(0.1)

            if not image_items:
                print("[WARNING] Image playlist is empty or not loaded.")
                continue

            # Case 1: We are NOT in image mode yet. Let's start it.
            if not auto_loop:
                print("[MODE] Entering Image Display Mode.")
                auto_loop = True
                if playlist_thread and playlist_thread.is_alive():
                    playlist_running = False
                    playlist_thread.join()
                
                image_index = 0
                display_image(image_items[image_index]['file'])
                last_switch_time = time.time()
            
            # Case 2: We are ALREADY in image mode. Just cycle to the next picture.
            else:
                print(f"[IMAGE] Cycling to next image.")
                image_index = (image_index + 1) % len(image_items)
                display_image(image_items[image_index]['file'])
                last_switch_time = time.time()

        elif GPIO.input(VIDEO_BUTTON) == GPIO.LOW:
            print("[BUTTON] VIDEO pressed")
            while GPIO.input(VIDEO_BUTTON) == GPIO.LOW: time.sleep(0.1)
            auto_loop = False
            if playlist_thread and playlist_thread.is_alive():
                playlist_running = False
                playlist_thread.join()
            items = load_playlist(VIDEO_PLAYLIST_FILE, 'video')
            if items:
                playlist_thread = threading.Thread(target=playlist_loop, args=(items,))
                playlist_thread.start()

        elif GPIO.input(PLAYLIST_BUTTON) == GPIO.LOW:
            print("[BUTTON] PLAYLIST pressed")
            while GPIO.input(PLAYLIST_BUTTON) == GPIO.LOW: time.sleep(0.1)
            auto_loop = False
            if playlist_thread and playlist_thread.is_alive():
                playlist_running = False
                playlist_thread.join()
            items = load_playlist(PLAYLIST_FILE)
            if items:
                playlist_thread = threading.Thread(target=playlist_loop, args=(items,))
                playlist_thread.start()

        if auto_loop and image_items:
            duration = image_items[image_index]['duration']
            if time.time() - last_switch_time >= duration:
                print("[IMAGE] Auto-advancing to next image.")
                image_index = (image_index + 1) % len(image_items)
                display_image(image_items[image_index]['file'])
                last_switch_time = time.time()

        time.sleep(0.1)

    # --- Cleanup ---
    print("[EXIT] Cleaning up...")
    if playlist_thread and playlist_thread.is_alive():
        playlist_running = False
        playlist_thread.join()
    GPIO.cleanup()
    pygame.quit()
    print("[EXIT] Done.")

if __name__ == "__main__":
    main()
