import pygame
import random
import sys

# Initialize Pygame
pygame.init()

# Constants
WIDTH, HEIGHT = 800, 600
FPS = 60
FRUIT_COUNT = 8
BOMB_IMAGE_PATH = 'bomb.png'  # Replace with the path to your bomb image
FRUIT_IMAGE_PATHS = [
    'apple.png',
    'banana.png',
    'orange.png',
    'grape.png',
    'avocado.png',
    'kiwi.png',
    'lemon.png',
    'papaya.png'
]
BACKGROUND_IMAGE_PATH = 'background.jpg'  # Replace with your background image path
BACKGROUND_MUSIC_PATH = 'rapgod.mp3'  # Replace with your music file path
SPLAT_SOUND_PATH = 'fruit slice.mp3'  # Replace with your sound file for slicing fruit
GAME_OVER_SOUND_PATH = 'Game Over.mp3'  # Replace with your game over sound file
IMAGE_SIZE = (64, 64)  # Original size for fruits and bomb
DROP_INTERVAL = 2000  # Drop interval for fruits in milliseconds (2 seconds)
BOMB_INTERVAL = 10000  # Bomb drop interval in milliseconds (10 seconds)
BASE_SPEED = 5  # Base falling speed for fruits and bombs

# Set up the display
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Fruit Slicer")

# Load images
bomb_image = pygame.image.load(BOMB_IMAGE_PATH)
bomb_image = pygame.transform.scale(bomb_image, IMAGE_SIZE)

fruit_images = []
for path in FRUIT_IMAGE_PATHS:
    fruit_image = pygame.image.load(path)
    fruit_image = pygame.transform.scale(fruit_image, IMAGE_SIZE)
    fruit_images.append(fruit_image)

# Load background image
background_image = pygame.image.load(BACKGROUND_IMAGE_PATH)
background_image = pygame.transform.scale(background_image, (WIDTH, HEIGHT))

# Load background music
pygame.mixer.music.load(BACKGROUND_MUSIC_PATH)
pygame.mixer.music.play(-1)  # Play indefinitely

# Load sound effects
splat_sound = pygame.mixer.Sound(SPLAT_SOUND_PATH)
game_over_sound = pygame.mixer.Sound(GAME_OVER_SOUND_PATH)

# Define speed levels based on score
def get_speed_multiplier(score):
    if score < 10:
        return 0.5  # Slow
    elif score < 20:
        return 0.8  # Slightly slow
    elif score < 30:
        return 1    # Normal speed
    elif score < 40:
        return 1.2  # Slightly fast
    elif score < 50:
        return 1.5  # Faster
    else:
        return 2    # Fastest

# Fruit class
class Fruit:
    def __init__(self, image, x, y, speed):
        self.image = image
        self.rect = self.image.get_rect(topleft=(x, y))
        self.sliced = False
        self.speed = speed

    def fall(self):
        self.rect.y += self.speed  # Move down at the current speed

    def draw(self, surface):
        if self.sliced:
            # Draw only the top half of the fruit
            top_half = self.image.subsurface((0, 0, self.rect.width, self.rect.height // 2))
            surface.blit(top_half, self.rect.topleft)
        else:
            surface.blit(self.image, self.rect)

    def slice(self):
        self.sliced = True

# Game loop
def main():
    clock = pygame.time.Clock()
    
    while True:  # Main game loop
        fruits = []
        bombs = []
        score = 0
        game_over = False
        last_fruit_drop_time = pygame.time.get_ticks()  # Timer for fruit drops
        last_bomb_drop_time = pygame.time.get_ticks()   # Timer for bomb drops

        # Generate initial fruits
        for _ in range(FRUIT_COUNT):
            fruit_type = random.choice(fruit_images)  # Select a random fruit
            x = random.randint(0, WIDTH - IMAGE_SIZE[0])
            fruits.append(Fruit(fruit_type, x, 0, BASE_SPEED))

        while not game_over:
            current_time = pygame.time.get_ticks()  # Get the current time

            # Update the speed multiplier based on score
            speed_multiplier = get_speed_multiplier(score)

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                    mouse_x, mouse_y = event.pos
                    for fruit in fruits:
                        if fruit.rect.collidepoint(mouse_x, mouse_y):
                            if fruit.image == bomb_image:
                                print("Game Over! You sliced a bomb!")
                                pygame.mixer.music.stop()  # Stop background music
                                game_over_sound.play()  # Play game over sound
                                game_over = True
                            else:
                                fruit.slice()  # Slice the fruit
                                score += 1
                                splat_sound.play()  # Play slicing sound
                                break  # Only slice one fruit at a time
                    # Check if the bomb is clicked
                    for bomb in bombs:
                        if bomb.rect.collidepoint(mouse_x, mouse_y):
                            print("Game Over! You sliced a bomb!")
                            pygame.mixer.music.stop()  # Stop background music
                            game_over_sound.play()  # Play game over sound
                            game_over = True

            if not game_over:
                for fruit in fruits:
                    fruit.speed = BASE_SPEED * speed_multiplier  # Adjust speed per level
                    fruit.fall()
                    if fruit.rect.top > HEIGHT:
                        fruits.remove(fruit)  # Remove fruits that fall out of the screen

                # Check if it's time to drop a new fruit
                if current_time - last_fruit_drop_time > DROP_INTERVAL:
                    fruit_type = random.choice(fruit_images)
                    x = random.randint(0, WIDTH - IMAGE_SIZE[0])
                    fruits.append(Fruit(fruit_type, x, 0, BASE_SPEED * speed_multiplier))
                    last_fruit_drop_time = current_time  # Reset the timer

                # Check if it's time to drop a bomb
                if current_time - last_bomb_drop_time > BOMB_INTERVAL:
                    x = random.randint(0, WIDTH - IMAGE_SIZE[0])
                    bombs.append(Fruit(bomb_image, x, 0, BASE_SPEED * speed_multiplier))  # Use the Fruit class for bombs
                    last_bomb_drop_time = current_time  # Reset the timer

                # Update bomb positions
                for bomb in bombs:
                    bomb.speed = BASE_SPEED * speed_multiplier  # Adjust speed per level
                    bomb.fall()
                    if bomb.rect.top > HEIGHT:
                        bombs.remove(bomb)  # Remove bombs that fall out of the screen

            # Clear the screen
            screen.blit(background_image, (0, 0))  # Draw the background

            # Draw fruits
            for fruit in fruits:
                fruit.draw(screen)

            # Draw bombs
            for bomb in bombs:
                bomb.draw(screen)

            # Draw score
            font = pygame.font.Font(None, 36)
            score_text = font.render(f'Score: {score}', True, (0, 0, 0))
            screen.blit(score_text, (10, 10))

            # Update the display
            pygame.display.flip()
            clock.tick(FPS)

        # Game over screen
        while game_over:
            screen.fill((0, 0, 0))  # Clear the screen
            font = pygame.font.Font(None, 74)
            game_over_text = font.render('Game Over!', True, (255, 0, 0))
            restart_text = font.render('Press R to Restart', True, (255, 255, 255))
            screen.blit(game_over_text, (WIDTH // 2 - 150, HEIGHT // 2 - 50))
            screen.blit(restart_text, (WIDTH // 2 - 200, HEIGHT // 2 + 20))
            pygame.display.flip()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                if event.type == pygame.KEYDOWN and event.key == pygame.K_r:
                    game_over = False  # Reset the game over state to start a new game
                    pygame.mixer.music.play(-1)  # Restart background music

if _name_ == "_main_":
    main()
