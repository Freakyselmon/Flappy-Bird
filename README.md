import pygame
import sys
import random

# Initialize Pygame
pygame.init()

# Screen dimensions
SCREEN_WIDTH = 400
SCREEN_HEIGHT = 600

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)

# Bird properties
BIRD_WIDTH = 40  # Adjust based on your bird image
BIRD_HEIGHT = 30  # Adjust based on your bird image
BIRD_X = 50
BIRD_Y = SCREEN_HEIGHT // 2
GRAVITY = 0.5
JUMP_STRENGTH = -10

# Pipe properties
PIPE_WIDTH = 60  # Pipe width
PIPE_HEIGHT = 400  # Height of the pipe image (smaller than screen height)
PIPE_GAP = 300  # Gap between upper and lower pipes
PIPE_SPEED = 3
PIPE_FREQUENCY = 1500  # Milliseconds

# Initialize screen
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Flappy Bird Clone")

# Clock for controlling frame rate
clock = pygame.time.Clock()

# Load images
try:
    bird_img = pygame.image.load("/Users/shabbirshaikh/Downloads/bird_-removebg-preview.png")  # Replace with your bird image
    bird_img = pygame.transform.scale(bird_img, (BIRD_WIDTH, BIRD_HEIGHT))

    pipe_img = pygame.image.load("/Users/shabbirshaikh/Downloads/pipe4-removebg-preview.png")  # Replace with your pipe image
    pipe_img = pygame.transform.scale(pipe_img, (PIPE_WIDTH, PIPE_HEIGHT))  # Scale pipe to new width and height

    background_img = pygame.image.load("/Users/shabbirshaikh/Downloads/background_and_road.png")  # Replace with your background image
    background_img = pygame.transform.scale(background_img, (SCREEN_WIDTH, SCREEN_HEIGHT))

    start_screen_img = pygame.image.load("/Users/shabbirshaikh/Downloads/flappy-bird-start-screen22.webp")  # Replace with your start screen image
    start_screen_img = pygame.transform.scale(start_screen_img, (SCREEN_WIDTH, SCREEN_HEIGHT))

    game_over_img = pygame.image.load("/Users/shabbirshaikh/Downloads/dies2.png")  # Replace with your game over image
    game_over_img = pygame.transform.scale(game_over_img, (SCREEN_WIDTH, SCREEN_HEIGHT))  # Scale game over image to screen size
except Exception as e:
    print("Error loading images:", e)
    sys.exit()

# Bird class
class Bird:
    def __init__(self):
        self.x = BIRD_X
        self.y = BIRD_Y
        self.velocity = 0
        self.width = BIRD_WIDTH
        self.height = BIRD_HEIGHT

    def jump(self):
        self.velocity = JUMP_STRENGTH

    def move(self):
        self.velocity += GRAVITY
        self.y += self.velocity

    def draw(self):
        screen.blit(bird_img, (self.x, self.y))

    def get_rect(self):
        return pygame.Rect(self.x, self.y, self.width, self.height)

# Pipe class
class Pipe:
    def __init__(self, x):
        self.x = x
        self.height = random.randint(100, 400)
        self.gap = PIPE_GAP
        self.width = PIPE_WIDTH
        self.passed = False

    def move(self):
        self.x -= PIPE_SPEED

    def draw(self):
        # Upper pipe (flipped vertically)
        upper_pipe = pygame.transform.flip(pipe_img, False, True)
        screen.blit(upper_pipe, (self.x, self.height - PIPE_HEIGHT))

        # Lower pipe
        screen.blit(pipe_img, (self.x, self.height + self.gap))

    def get_rects(self):
        return [
            pygame.Rect(self.x, self.height - PIPE_HEIGHT, self.width, PIPE_HEIGHT),  # Upper pipe
            pygame.Rect(self.x, self.height + self.gap, self.width, PIPE_HEIGHT)  # Lower pipe
        ]

# Function to read high score from file
def read_high_score():
    try:
        with open("high_score.txt", "r") as file:
            return int(file.read())
    except FileNotFoundError:
        return 0

# Function to write high score to file
def write_high_score(score):
    with open("high_score.txt", "w") as file:
        file.write(str(score))

# Game variables
bird = Bird()
pipes = []
score = 0
high_score = read_high_score()
last_pipe_time = pygame.time.get_ticks()
game_over = False
game_started = False  # Track if the game has started

# Font for displaying score
font = pygame.font.SysFont("Arial", 30)
large_font = pygame.font.SysFont("Arial", 50, bold=True)  # Larger font for final score

# Function to display the start screen
def show_start_screen():
    screen.blit(start_screen_img, (0, 0))
    pygame.display.update()

# Function to display the game over screen
def show_game_over_screen():
    screen.blit(game_over_img, (0, 0))  # Display game over image

    # Display final score in big bold letters
    final_score_text = large_font.render(f"Score: {score}", True, WHITE)
    # Calculate text position for centering
    text_width, text_height = large_font.size(f"Score: {score}")
    text_x = (SCREEN_WIDTH - text_width) // 2
    text_y = (SCREEN_HEIGHT - text_height) // 2 - 50
    screen.blit(final_score_text, (text_x, text_y))

    # Display high score in big bold letters
    high_score_text = large_font.render(f"High Score: {high_score}", True, WHITE)
    text_width, text_height = large_font.size(f"High Score: {high_score}")
    text_x = (SCREEN_WIDTH - text_width) // 2
    text_y = (SCREEN_HEIGHT - text_height) // 2 + 50
    screen.blit(high_score_text, (text_x, text_y))

    pygame.display.update()

# Main game loop
while True:
    # Event handling
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE and not game_over and game_started:
                bird.jump()
            if event.key == pygame.K_r and game_over:
                # Reset game
                bird = Bird()
                pipes = []
                score = 0
                game_over = False
                game_started = True
        if event.type == pygame.MOUSEBUTTONDOWN and not game_started:
            # Start the game when the player clicks anywhere on the screen
            game_started = True

    if not game_started:
        # Show the start screen
        show_start_screen()
        continue  # Skip the rest of the loop until the game starts

    if not game_over:
        # Bird movement
        bird.move()

        # Pipe spawning
        current_time = pygame.time.get_ticks()
        if current_time - last_pipe_time > PIPE_FREQUENCY:
            pipes.append(Pipe(SCREEN_WIDTH))
            last_pipe_time = current_time

        # Pipe movement and collision detection
        for pipe in pipes:
            pipe.move()
            for rect in pipe.get_rects():
                if bird.get_rect().colliderect(rect):
                    print("Collision detected!")  # Debug statement
                    game_over = True
            if pipe.x + pipe.width < bird.x and not pipe.passed:
                pipe.passed = True
                score += 1

        # Remove off-screen pipes
        pipes = [pipe for pipe in pipes if pipe.x + pipe.width > 0]

        # Check if bird hits the ground
        if bird.y + bird.height > SCREEN_HEIGHT:
            print("Bird hit the ground!")  # Debug statement
            game_over = True

        # Update high score if current score is higher
        if score > high_score:
            high_score = score
            write_high_score(high_score)

    # Drawing
    if not game_over:
        screen.blit(background_img, (0, 0))  # Draw background
        bird.draw()
        for pipe in pipes:
            pipe.draw()

        # Display score in black
        score_text = font.render(f"Score: {score}", True, BLACK)
        screen.blit(score_text, (10, 10))

    # Display game over screen if the game is over
    if game_over:
        show_game_over_screen()

    # Update display
    pygame.display.update()
    clock.tick(30)
