This is a game of left hand versus right hand, let's see which of your two hands is smarter.
This program still has many imperfections. Please help point them out and assist in fixing them. Thank you.
python code



import pygame
import sys
import random

# Initialize Pygame
pygame.init()

# Set window size and title
WINDOW_WIDTH = 800
WINDOW_HEIGHT = 600
WINDOW = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption("Ping Pong Game")

# Define colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)

# Define paddles
class Paddle(pygame.sprite.Sprite):
    def __init__(self, x, y, control_keys):
        super().__init__()
        self.image = pygame.Surface((20, 100))
        self.image.fill(WHITE)
        self.rect = self.image.get_rect()
        self.rect.center = (x, y)
        self.control_keys = control_keys

    def update(self, keys):
        if keys[self.control_keys[0]]:
            self.rect.y -= 5
        if keys[self.control_keys[1]]:
            self.rect.y += 5

        # Prevent paddle from moving out of the window
        if self.rect.top < 0:
            self.rect.top = 0
        if self.rect.bottom > WINDOW_HEIGHT:
            self.rect.bottom = WINDOW_HEIGHT

# Define ball
class Ball(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = pygame.Surface((20, 20))
        self.image.fill(WHITE)
        self.rect = self.image.get_rect()
        self.rect.center = (WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2)
        self.speed_x = 5
        self.speed_y = 5
        self.initial_speed = (self.speed_x, self.speed_y)

    def update(self):
        self.rect.x += self.speed_x
        self.rect.y += self.speed_y

        # Bounce off the top and bottom edges
        if self.rect.top <= 0 or self.rect.bottom >= WINDOW_HEIGHT:
            self.speed_y *= -1

        # Bounce off the paddles
        if pygame.sprite.spritecollide(self, paddles, False):
            self.speed_x *= -1

        # Bounce off the left and right edges
        if self.rect.left <= 0 or self.rect.right >= WINDOW_WIDTH:
            self.speed_x *= -1

        # Slow down the ball when it first enters the window
        if self.rect.left <= 0:
            self.rect.left = 0
            self.speed_x, self.speed_y = self.initial_speed
            self.speed_x *= 0.5

        elif self.rect.right >= WINDOW_WIDTH:
            self.rect.right = WINDOW_WIDTH
            self.speed_x, self.speed_y = self.initial_speed
            self.speed_x *= 0.5

# Initialize the game
def initialize_game():
    global paddles, ball, clock, font, score_left, score_right
    paddles = pygame.sprite.Group()
    paddle_left = Paddle(50, WINDOW_HEIGHT // 2, (pygame.K_w, pygame.K_s))
    paddle_right = Paddle(WINDOW_WIDTH - 50, WINDOW_HEIGHT // 2, (pygame.K_o, pygame.K_k))
    paddles.add(paddle_left, paddle_right)

    ball = Ball()

    clock = pygame.time.Clock()

    font = pygame.font.Font(None, 36)

    score_left = 0
    score_right = 0

# Display text
def display_text(text, x, y):
    text_surface = font.render(text, True, WHITE)
    text_rect = text_surface.get_rect()
    text_rect.center = (x, y)
    WINDOW.blit(text_surface, text_rect)

# Main game loop
def main():
    global ball, score_left, score_right

    initialize_game()

    running = True
    game_start_time = pygame.time.get_ticks()
    ball_speed_timer = pygame.time.get_ticks()

    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

        keys = pygame.key.get_pressed()
        paddles.update(keys)
        ball.update()

        # Check score
        if ball.rect.left <= 0:
            score_right += 1
            ball.rect.center = (WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2)
            ball_speed_timer = pygame.time.get_ticks()
        elif ball.rect.right >= WINDOW_WIDTH:
            score_left += 1
            ball.rect.center = (WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2)
            ball_speed_timer = pygame.time.get_ticks()

        # Control ball speed
        if pygame.time.get_ticks() - ball_speed_timer >= 15000:
            ball.speed_x *= 1.1
            ball.speed_y *= 1.1
            ball_speed_timer = pygame.time.get_ticks()

        # Check game end
        if pygame.time.get_ticks() - game_start_time >= 300000:
            running = False

        # Render
        WINDOW.fill(BLACK)
        display_text("Left: {}".format(score_left), 100, 50)
        display_text("Right: {}".format(score_right), WINDOW_WIDTH - 100, 50)
        paddles.draw(WINDOW)
        pygame.draw.ellipse(WINDOW, WHITE, ball.rect)
        pygame.display.flip()

        clock.tick(60)

    # Display game result
    if score_left > score_right:
        display_text("Left Wins", WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2)
    elif score_right > score_left:
        display_text("Right Wins", WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2)
    else:
        display_text("Draw", WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2)

    pygame.display.flip()
    pygame.time.wait(3000)
    pygame.quit()
    sys.exit()

if __name__ == "__main__":
    main()
