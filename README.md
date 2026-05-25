# =========================================
# PING PONG
# =========================================

import pygame
import sys
import random

# =========================================
# ИНИЦИАЛИЗАЦИЯ
# =========================================

pygame.init()
pygame.mixer.init()

# =========================================
# ЗВУКИ
# =========================================

def load_sound(filename):

    try:
        sound = pygame.mixer.Sound(filename)
        sound.set_volume(0.3)
        return sound

    except pygame.error as e:
        print(f"Ошибка загрузки звука {filename}: {e}")
        return None

bounce_sound = load_sound('assets/sounds/otskok_n.mp3')
score_sound = load_sound('assets/sounds/ochko.mp3')
win_sound = load_sound('assets/sounds/win.mp3')

# =========================================
# ЭКРАН
# =========================================

screen = pygame.display.set_mode((0, 0), pygame.FULLSCREEN)

WIDTH, HEIGHT = screen.get_size()

pygame.display.set_caption("Ping-pong")

# =========================================
# РЕЖИМЫ
# =========================================

SINGLE_PLAYER = 1
TWO_PLAYERS = 2

# =========================================
# ЦВЕТА
# =========================================

BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
BLUE = (60, 185, 243)

# =========================================
# РАКЕТКИ
# =========================================

PADDLE_WIDTH = 30
PADDLE_HEIGHT = 150

PADDLE_SPEED = 8

paddle1_y = HEIGHT // 2 - PADDLE_HEIGHT // 2
paddle2_y = HEIGHT // 2 - PADDLE_HEIGHT // 2

# =========================================
# МЯЧ
# =========================================

try:

    original_ball_image = pygame.image.load(
        'assets/Ballwithout.png'
    )

except FileNotFoundError:

    print("Мяч не найден")

    original_ball_image = pygame.Surface((60, 60))
    original_ball_image.fill(WHITE)

TARGET_BALL_WIDTH = 50
TARGET_BALL_HEIGHT = 50

ball_image = pygame.transform.scale(
    original_ball_image,
    (TARGET_BALL_WIDTH, TARGET_BALL_HEIGHT)
)

BALL_WIDTH, BALL_HEIGHT = ball_image.get_size()

# =========================================
# ШРИФТЫ
# =========================================

try:

    menu_font = pygame.font.Font(
        "assets/fonts/LCD16x2Remastered-RegularV2.otf",
        74
    )

    font = pygame.font.Font(
        "assets/fonts/LCD16x2Remastered-RegularV2.otf",
        44
    )

except FileNotFoundError:

    print("Шрифт не найден")

    menu_font = pygame.font.Font(None, 74)
    font = pygame.font.Font(None, 44)

# =========================================
# ФОН ТЕКСТА
# =========================================

try:

    menu_text_bg = pygame.image.load('assets/cloud.png')

    menu_text_bg = pygame.transform.scale(
        menu_text_bg,
        (450, 200)
    )

except FileNotFoundError:

    print("Фон текста не найден")

    menu_text_bg = None

# =========================================
# ФОН КНОПОК
# =========================================

try:
    button_background_img = pygame.image.load('assets/cloud.png')

except FileNotFoundError:

    print("Фон кнопок не найден")

    button_background_img = None

# =========================================
# ОБЛАКА
# =========================================

try:
    cloud_img = pygame.image.load('assets/cloud_back.png')

except FileNotFoundError:

    print("Облака не найдены")

    cloud_img = None

# =========================================
# КАРТИНКА РАКЕТКИ
# =========================================

try:

    paddle_img = pygame.image.load('assets/paddle.png')

    paddle_img = pygame.transform.scale(
        paddle_img,
        (PADDLE_WIDTH, PADDLE_HEIGHT)
    )

except FileNotFoundError:

    paddle_img = None

# =========================================
# ВЕРХНЕЕ ОБЛАКО
# =========================================

CLOUD_WALL_HEIGHT = 100

try:

    top_wall_cloud = pygame.image.load('assets/cloud.png')

    top_wall_cloud = pygame.transform.scale(
        top_wall_cloud,
        (WIDTH, CLOUD_WALL_HEIGHT)
    )

except FileNotFoundError:

    print("Верхнее облако не найдено")

    top_wall_cloud = None

# =========================================
# ДОМИК
# =========================================

try:

    home_icon_original = pygame.image.load(
        'assets/home_icon.png'
    )

    HOME_ICON_SIZE = 50

    home_icon = pygame.transform.scale(
        home_icon_original,
        (HOME_ICON_SIZE, HOME_ICON_SIZE)
    )

    home_icon = home_icon.copy()

    home_icon.fill(
        BLUE,
        special_flags=pygame.BLEND_RGB_MULT
    )

    home_icon_rect = home_icon.get_rect(
        centerx=WIDTH // 2,
        centery=CLOUD_WALL_HEIGHT // 1.5
    )

except FileNotFoundError:

    print("Домик не найден")

    home_icon = None
    home_icon_rect = pygame.Rect(0, 0, 0, 0)

# =========================================
# ОБЛАКА КЛАСС
# =========================================

class Cloud:

    def __init__(self, x, y, speed, scale):

        self.x = x
        self.y = y
        self.speed = speed
        self.scale = scale

        self.image = pygame.transform.scale(
            cloud_img,
            (
                int(cloud_img.get_width() * scale),
                int(cloud_img.get_height() * scale)
            )
        )

        self.width = self.image.get_width()

    def update(self):

        self.x -= self.speed

        if self.x < -self.width:

            self.x = WIDTH + random.randint(1, 200)

            self.y = random.randint(50, HEIGHT - 150)

    def draw(self, screen):

        screen.blit(self.image, (self.x, self.y))

# =========================================
# СОЗДАНИЕ ОБЛАКОВ
# =========================================

clouds = []

if cloud_img:

    for _ in range(5):

        x = WIDTH + random.randint(0, WIDTH)

        y = random.randint(50, HEIGHT - 150)

        speed = random.uniform(0.5, 2.0)

        scale = random.uniform(1, 10)

        clouds.append(
            Cloud(x, y, speed, scale)
        )

clock = pygame.time.Clock()

# =========================================
# СЧЁТ
# =========================================

score1 = 0
score2 = 0

# =========================================
# МЯЧ
# =========================================

ball_x = WIDTH // 2 - BALL_WIDTH // 2
ball_y = HEIGHT // 2 - BALL_HEIGHT // 2

# =========================================
# СКОРОСТЬ
# =========================================

ball_speed = 5

ball_dx = random.choice([-ball_speed, ball_speed])
ball_dy = random.choice([-ball_speed, ball_speed])

# =========================================
# УСКОРЕНИЕ
# =========================================

ricochet_count = 0

SPEED_INCREASE_PER_RICOCHET = 0.15

MAX_SPEED_MULTIPLIER = 1.5

WINNING_SCORE = 5

# =========================================
# КНОПКИ
# =========================================

def draw_button(text, x, y, width, height, hover=False):

    if button_background_img:

        scaled_bg = pygame.transform.scale(
            button_background_img,
            (width, height)
        )

        screen.blit(scaled_bg, (x, y))

    text_surf = font.render(text, True, BLUE)

    text_rect = text_surf.get_rect(
        center=(x + width // 2, y + height // 2)
    )

    screen.blit(text_surf, text_rect)

    return pygame.Rect(x, y, width, height)

# =========================================
# МЕНЮ
# =========================================

def draw_menu():

    screen.fill(BLUE)

    for cloud in clouds:

        cloud.update()
        cloud.draw(screen)

    text_bg_x = WIDTH // 2 - 225
    text_bg_y = HEIGHT // 4 - 50

    if menu_text_bg:

        screen.blit(
            menu_text_bg,
            (text_bg_x, text_bg_y)
        )

    else:

        s = pygame.Surface((400, 120), pygame.SRCALPHA)

        s.fill((60, 185, 243, 180))

        screen.blit(s, (text_bg_x, text_bg_y))

    menu_text = menu_font.render(
        "МЕНЮ",
        True,
        BLUE
    )

    text_x = WIDTH // 2 - menu_text.get_width() // 2
    text_y = HEIGHT // 4

    screen.blit(menu_text, (text_x, text_y))

    button_width = 510
    button_height = 120

    single_y = HEIGHT // 2
    two_y = single_y + 120
    exit_y = two_y + 120

    single_rect = draw_button(
        "ОДИНОЧНЫЙ",
        WIDTH // 2 - button_width // 2,
        single_y,
        button_width,
        button_height
    )

    two_rect = draw_button(
        "ДВОЙНОЙ",
        WIDTH // 2 - button_width // 2,
        two_y,
        button_width,
        button_height
    )

    exit_rect = draw_button(
        "ВЫХОД",
        WIDTH // 2 - button_width // 2,
        exit_y,
        button_width,
        button_height
    )

    return single_rect, two_rect, exit_rect

# =========================================
# ПОКАЗ МЕНЮ
# =========================================

def show_menu():

    while True:

        single_rect, two_rect, exit_rect = draw_menu()

        for event in pygame.event.get():

            if event.type == pygame.QUIT:

                pygame.quit()
                sys.exit()

            elif event.type == pygame.KEYDOWN:

                if event.key == pygame.K_ESCAPE:

                    pygame.quit()
                    sys.exit()

            elif event.type == pygame.MOUSEBUTTONDOWN:

                if event.button == 1:

                    if single_rect.collidepoint(event.pos):
                        return SINGLE_PLAYER

                    elif two_rect.collidepoint(event.pos):
                        return TWO_PLAYERS

                    elif exit_rect.collidepoint(event.pos):

                        pygame.quit()
                        sys.exit()

        pygame.display.flip()
        clock.tick(60)


def show_difficulty_menu():
    while True:
        screen.fill(BLUE)

        # Рисуем анимированные облака на фоне
        for cloud in clouds:
            cloud.update()
            cloud.draw(screen)

        # Параметры увеличенного облачка под текст (было 400×80, теперь 500×100)
        cloud_bg_width = 550
        cloud_bg_height = 150
        cloud_bg_x = WIDTH // 2 - cloud_bg_width // 2
        cloud_bg_y = HEIGHT // 4 - cloud_bg_height // 2

        if menu_text_bg:
            # Масштабируем исходное изображение облака под новые размеры
            scaled_cloud_bg = pygame.transform.scale(menu_text_bg, (cloud_bg_width, cloud_bg_height))
            screen.blit(scaled_cloud_bg, (cloud_bg_x, cloud_bg_y))
        else:
            # Альтернативный фон, если картинка не загрузилась — полупрозрачный прямоугольник
            s = pygame.Surface((cloud_bg_width, cloud_bg_height), pygame.SRCALPHA)
            s.fill((60, 185, 243, 180))  # Синий с прозрачностью
            screen.blit(s, (cloud_bg_x, cloud_bg_y))

        # Шрифт для заголовка
        try:
            small_font = pygame.font.Font("assets/fonts/LCD16x2Remastered-RegularV2.otf", 36)
        except:
            small_font = pygame.font.Font(None, 36)  # запасной системный шрифт

        # Отрисовываем текст «СЛОЖНОСТЬ», центрируем по облачку
        title = small_font.render("СЛОЖНОСТЬ", True, BLUE)
        title_rect = title.get_rect(center=(WIDTH // 2, HEIGHT // 4))
        screen.blit(title, title_rect)

        # Параметры кнопок
        button_width = 510
        button_height = 120
        easy_y = HEIGHT // 2
        medium_y = easy_y + 120
        hard_y = medium_y + 120

        # Рисуем кнопки и получаем их прямоугольники для кликов
        easy_rect = draw_button("ЛЕГКО", WIDTH // 2 - button_width // 2, easy_y, button_width, button_height)
        medium_rect = draw_button("СРЕДНЕ", WIDTH // 2 - button_width // 2, medium_y, button_width, button_height)
        hard_rect = draw_button("СЛОЖНО", WIDTH // 2 - button_width // 2, hard_y, button_width, button_height)

        # Обработка событий
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_ESCAPE:
                    pygame.quit()
                    sys.exit()
            elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                if easy_rect.collidepoint(event.pos):
                    return 4
                elif medium_rect.collidepoint(event.pos):
                    return 7
                elif hard_rect.collidepoint(event.pos):
                    return 11

        pygame.display.flip()
        clock.tick(60)



# =========================================
# ВВОД ОЧКОВ
# =========================================

def show_score_input():

    user_input = ""

    prompt_text = font.render(
        "КОЛ-ВО ОЧКОВ ДЛЯ ПОБЕДЫ:",
        True,
        BLUE
    )

    instruction_text = font.render(
        "НАЖМИТЕ ENTER ДЛЯ ВВОДА",
        True,
        BLUE
    )

    while True:

        for event in pygame.event.get():

            if event.type == pygame.QUIT:

                pygame.quit()
                sys.exit()

            elif event.type == pygame.KEYDOWN:

                if event.key == pygame.K_RETURN:

                    if user_input.isdigit() and int(user_input) > 0:
                        return int(user_input)

                elif event.key == pygame.K_BACKSPACE:

                    user_input = user_input[:-1]

                else:

                    if len(user_input) < 3:
                        user_input += event.unicode

        screen.fill(WHITE)

        prompt_rect = prompt_text.get_rect(
            center=(WIDTH // 2, HEIGHT // 2.5 - 80)
        )

        instruction_rect = instruction_text.get_rect(
            center=(WIDTH // 2, HEIGHT // 1.5 + 80)
        )

        screen.blit(prompt_text, prompt_rect)
        screen.blit(instruction_text, instruction_rect)

        input_rect = pygame.Rect(
            WIDTH // 2 - 100,
            HEIGHT // 2 - 30,
            200,
            100
        )

        pygame.draw.rect(
            screen,
            WHITE,
            input_rect
        )

        pygame.draw.rect(
            screen,
            BLUE,
            input_rect,
            3
        )

        input_display = font.render(
            user_input,
            True,
            BLUE
        )

        input_display_rect = input_display.get_rect(
            center=input_rect.center
        )

        screen.blit(
            input_display,
            input_display_rect
        )

        pygame.display.flip()
        clock.tick(60)

# =========================================
# ИИ
# =========================================

def ai_move(paddle_y, current_ball_y):

    if paddle_y + PADDLE_HEIGHT // 2 < current_ball_y:

        return min(
            paddle_y + PADDLE_SPEED,
            HEIGHT - PADDLE_HEIGHT
        )

    elif paddle_y + PADDLE_HEIGHT // 2 > current_ball_y:

        return max(
            paddle_y - PADDLE_SPEED,
            CLOUD_WALL_HEIGHT
        )

    return paddle_y

# =========================================
# GAME OVER
# =========================================

def show_game_over(score1, score2):

    screen.fill(WHITE)

    if win_sound:
        win_sound.play()

    game_over_text = menu_font.render(
        "ИГРА ОКОНЧЕНА",
        True,
        BLUE
    )

    game_over_rect = game_over_text.get_rect(
        center=(WIDTH // 2, HEIGHT // 3)
    )

    screen.blit(game_over_text, game_over_rect)

    if score1 > score2:

        winner_text = menu_font.render(
            "Игрок 1 победил!",
            True,
            BLUE
        )

    else:

        winner_text = menu_font.render(
            "Игрок 2 победил!",
            True,
            BLUE
        )

    winner_rect = winner_text.get_rect(
        center=(WIDTH // 2, HEIGHT // 2)
    )

    screen.blit(winner_text, winner_rect)

    pygame.display.flip()

    pygame.time.wait(3000)

# =========================================
# ОСНОВНОЙ ЦИКЛ
# =========================================

while True:

    # Меню
    game_mode = show_menu()

    # Сложность
    ball_speed = show_difficulty_menu()

    # Очки
    WINNING_SCORE = show_score_input()

    # Сброс
    score1 = 0
    score2 = 0

    ricochet_count = 0

    paddle1_y = HEIGHT // 2 - PADDLE_HEIGHT // 2
    paddle2_y = HEIGHT // 2 - PADDLE_HEIGHT // 2

    ball_x = WIDTH // 2 - BALL_WIDTH // 2
    ball_y = HEIGHT // 2 - BALL_HEIGHT // 2

    ball_dx = random.choice([-ball_speed, ball_speed])
    ball_dy = random.choice([-ball_speed, ball_speed])

    running = True

    while running:

        # СОБЫТИЯ
        for event in pygame.event.get():

            if event.type == pygame.QUIT:

                pygame.quit()
                sys.exit()

            elif event.type == pygame.KEYDOWN:

                if event.key == pygame.K_ESCAPE:

                    pygame.quit()
                    sys.exit()

            elif event.type == pygame.MOUSEBUTTONDOWN:

                if event.button == 1:

                    if home_icon and home_icon_rect.collidepoint(event.pos):

                        running = False

        # УПРАВЛЕНИЕ
        keys = pygame.key.get_pressed()

        if game_mode == SINGLE_PLAYER:
            # Одиночный режим: только правая платформа, стрелки вверх/вниз
            if keys[pygame.K_UP]:
                paddle1_y = max(paddle1_y - PADDLE_SPEED, CLOUD_WALL_HEIGHT)
            if keys[pygame.K_DOWN]:
                paddle1_y = min(paddle1_y + PADDLE_SPEED, HEIGHT - PADDLE_HEIGHT)
        else:  # TWO_PLAYERS
            # Многопользовательский режим: левая платформа — W/S, правая — стрелки
            if keys[pygame.K_w]:
                paddle1_y = max(paddle1_y - PADDLE_SPEED, CLOUD_WALL_HEIGHT)
            if keys[pygame.K_s]:
                paddle1_y = min(paddle1_y + PADDLE_SPEED, HEIGHT - PADDLE_HEIGHT)
            if keys[pygame.K_UP]:
                paddle2_y = max(paddle2_y - PADDLE_SPEED, CLOUD_WALL_HEIGHT)
            if keys[pygame.K_DOWN]:
                paddle2_y = min(paddle2_y + PADDLE_SPEED, HEIGHT - PADDLE_HEIGHT)

        # В одиночном режиме левая платформа управляется ИИ
        if game_mode == SINGLE_PLAYER:
            paddle2_y = ai_move(paddle2_y, ball_y)

        # =========================================
        # ДВИЖЕНИЕ МЯЧА
        # =========================================

        ball_x += ball_dx
        ball_y += ball_dy

        # ВЕРХ
        if ball_y <= CLOUD_WALL_HEIGHT:

            ball_dy *= -1

            if bounce_sound:
                bounce_sound.play()

        # НИЗ
        elif ball_y >= HEIGHT - BALL_HEIGHT:

            ball_dy *= -1

            if bounce_sound:
                bounce_sound.play()

        # =========================================
        # СТОЛКНОВЕНИЯ
        # =========================================

        paddle1_rect = pygame.Rect(
            0,
            paddle1_y,
            PADDLE_WIDTH,
            PADDLE_HEIGHT
        )

        paddle2_rect = pygame.Rect(
            WIDTH - PADDLE_WIDTH,
            paddle2_y,
            PADDLE_WIDTH,
            PADDLE_HEIGHT
        )

        ball_rect = pygame.Rect(
            ball_x,
            ball_y,
            BALL_WIDTH,
            BALL_HEIGHT
        )

        # ЛЕВАЯ РАКЕТКА
        if ball_rect.colliderect(paddle1_rect):

            ball_x = PADDLE_WIDTH

            ball_dx *= -1

            ricochet_count += 1

            if bounce_sound:
                bounce_sound.play()

            current_speed_multiplier = 1 + (
                ricochet_count * SPEED_INCREASE_PER_RICOCHET
            )

            if current_speed_multiplier > MAX_SPEED_MULTIPLIER:

                current_speed_multiplier = MAX_SPEED_MULTIPLIER

            ball_dx *= current_speed_multiplier
            ball_dy *= current_speed_multiplier

        # ПРАВАЯ РАКЕТКА
        elif ball_rect.colliderect(paddle2_rect):

            ball_x = WIDTH - PADDLE_WIDTH - BALL_WIDTH

            ball_dx *= -1

            ricochet_count += 1

            if bounce_sound:
                bounce_sound.play()

            current_speed_multiplier = 1 + (
                ricochet_count * SPEED_INCREASE_PER_RICOCHET
            )

            if current_speed_multiplier > MAX_SPEED_MULTIPLIER:

                current_speed_multiplier = MAX_SPEED_MULTIPLIER

            ball_dx *= current_speed_multiplier
            ball_dy *= current_speed_multiplier

        # =========================================
        # ГОЛЫ
        # =========================================

        if ball_x <= 0:

            score2 += 1

            if score_sound:
                score_sound.play()

            ball_x = WIDTH // 2 - BALL_WIDTH // 2
            ball_y = HEIGHT // 2 - BALL_HEIGHT // 2

            ball_dx = random.choice([-ball_speed, ball_speed])
            ball_dy = random.choice([-ball_speed, ball_speed])

            ricochet_count = 0

        elif ball_x >= WIDTH - BALL_WIDTH:

            score1 += 1

            if score_sound:
                score_sound.play()

            ball_x = WIDTH // 2 - BALL_WIDTH // 2
            ball_y = HEIGHT // 2 - BALL_HEIGHT // 2

            ball_dx = random.choice([-ball_speed, ball_speed])
            ball_dy = random.choice([-ball_speed, ball_speed])

            ricochet_count = 0

        # =========================================
        # ПОБЕДА
        # =========================================

        if score1 >= WINNING_SCORE:

            show_game_over(score1, score2)

            running = False

        elif score2 >= WINNING_SCORE:

            show_game_over(score1, score2)

            running = False

        # =========================================
        # ОТРИСОВКА
        # =========================================

        screen.fill(BLUE)

        # Верхнее облако
        if top_wall_cloud:

            screen.blit(top_wall_cloud, (0, 0))

        else:

            pygame.draw.rect(
                screen,
                BLUE,
                (0, 0, WIDTH, CLOUD_WALL_HEIGHT)
            )

        # Левая ракетка
        if paddle_img:

            screen.blit(
                paddle_img,
                (0, paddle1_y)
            )

        else:

            pygame.draw.rect(
                screen,
                WHITE,
                paddle1_rect
            )

        # Правая ракетка
        if paddle_img:

            flipped_paddle = pygame.transform.flip(
                paddle_img,
                True,
                False
            )

            screen.blit(
                flipped_paddle,
                (WIDTH - PADDLE_WIDTH, paddle2_y)
            )

        else:

            pygame.draw.rect(
                screen,
                WHITE,
                paddle2_rect
            )

        # Мяч
        screen.blit(ball_image, (ball_x, ball_y))

        # Счёт
        score_text = font.render(
            f"{score1}  {score2}",
            True,
            BLUE
        )

        screen.blit(
            score_text,
            (
                WIDTH // 2 - score_text.get_width() // 2,
                20
            )
        )

        # Домик
        if home_icon:

            screen.blit(
                home_icon,
                home_icon_rect.topleft
            )

        pygame.display.flip()

        clock.tick(60)


pygame.quit()
sys.exit()
