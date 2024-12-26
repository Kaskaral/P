import pygame
import random

# Инициализация Pygame
pygame.init()

# Настройки экрана
WIDTH, HEIGHT = 300, 600
BLOCK_SIZE = 30
GRID_WIDTH, GRID_HEIGHT = WIDTH // BLOCK_SIZE, HEIGHT // BLOCK_SIZE
FPS = 10

# Цвета
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GRAY = (50, 50, 50)
COLORS = [
    (255, 0, 0), (0, 255, 0), (0, 0, 255),
    (255, 255, 0), (255, 165, 0), (0, 255, 255)
]

# Фигуры
SHAPES = [
    [[1, 1, 1], [0, 1, 0]],  # T-образная
    [[1, 1, 0], [0, 1, 1]],  # Z-образная
    [[0, 1, 1], [1, 1, 0]],  # S-образная
    [[1, 1, 1, 1]],          # Прямая
    [[1, 1], [1, 1]],        # Квадрат
    [[1, 1, 1], [1, 0, 0]],  # L-образная
    [[1, 1, 1], [0, 0, 1]]   # Обратная L
]

# Функция для вращения фигур
def rotate_shape(shape):
    return [list(row) for row in zip(*shape[::-1])]

# Класс игры
class Tetris:
    def init(self):
        self.screen = pygame.display.set_mode((WIDTH, HEIGHT))
        pygame.display.set_caption("Tetris")
        self.clock = pygame.time.Clock()
        self.grid = [[0] * GRID_WIDTH for _ in range(GRID_HEIGHT)]
        self.current_shape = random.choice(SHAPES)
        self.current_color = random.choice(COLORS)
        self.shape_x, self.shape_y = GRID_WIDTH // 2 - len(self.current_shape[0]) // 2, 0
        self.score = 0

    def draw_grid(self):
        for x in range(0, WIDTH, BLOCK_SIZE):
            pygame.draw.line(self.screen, GRAY, (x, 0), (x, HEIGHT))
        for y in range(0, HEIGHT, BLOCK_SIZE):
            pygame.draw.line(self.screen, GRAY, (0, y), (WIDTH, y))

    def draw_shape(self):
        for y, row in enumerate(self.current_shape):
            for x, cell in enumerate(row):
                if cell:
                    pygame.draw.rect(
                        self.screen,
                        self.current_color,
                        pygame.Rect(
                            (self.shape_x + x) * BLOCK_SIZE,
                            (self.shape_y + y) * BLOCK_SIZE,
                            BLOCK_SIZE,
                            BLOCK_SIZE
                        )
                    )

    def draw_grid_blocks(self):
        for y, row in enumerate(self.grid):
            for x, cell in enumerate(row):
                if cell:
                    pygame.draw.rect(
                        self.screen,
                        cell,
                        pygame.Rect(
                            x * BLOCK_SIZE,
                            y * BLOCK_SIZE,
                            BLOCK_SIZE,
                            BLOCK_SIZE
                        )
                    )

    def can_move(self, dx, dy, shape=None):
        shape = shape or self.current_shape
        for y, row in enumerate(shape):
            for x, cell in enumerate(row):
                if cell:
                    nx, ny = self.shape_x + x + dx, self.shape_y + y + dy
                    if nx < 0 or nx >= GRID_WIDTH or ny >= GRID_HEIGHT or (ny >= 0 and self.grid[ny][nx]):
                        return False
        return True

    def merge_shape(self):
        for y, row in enumerate(self.current_shape):
            for x, cell in enumerate(row):
                if cell:
                    self.grid[self.shape_y + y][self.shape_x + x] = self.current_color

    def clear_lines(self):
        new_grid = [row for row in self.grid if any(cell == 0 for cell in row)]
        cleared_lines = GRID_HEIGHT - len(new_grid)
        self.grid = [[0] * GRID_WIDTH for _ in range(cleared_lines)] + new_grid
        self.score += cleared_lines * 10

    def next_shape(self):
        self.current_shape = random.choice(SHAPES)
        self.current_color = random.choice(COLORS)
        self.shape_x = GRID_WIDTH // 2 - len(self.current_shape[0]) // 2
        self.shape_y = 0
        if not self.can_move(0, 0):
            return False
        return True

    def run(self):
        running = True
        while running:
            self.screen.fill(BLACK)
            self.draw_grid()
            self.draw_grid_blocks()
            self.draw_shape()
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_LEFT and self.can_move(-1, 0):
                        self.shape_x -= 1
                    if event.key == pygame.K_RIGHT and self.can_move(1, 0):
                        self.shape_x += 1
                    if event.key == pygame.K_DOWN and self.can_move(0, 1):
                        self.shape_y += 1
                    if event.key == pygame.K_UP:
                        rotated_shape = rotate_shape(self.current_shape)
                        if self.can_move(0, 0, rotated_shape):
                            self.current_shape = rotated_shape

            if not self.can_move(0, 1):
                self.merge_shape()
                self.clear_lines()
                if not self.next_shape():
                    print("Game Over! Your score:", self.score)
                    running = False
            else:
                self.shape_y += 1

            pygame.display.flip()
            self.clock.tick(FPS)

# Запуск игры
if name == "main":
    game = Tetris()
    game.run()
