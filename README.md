import pygame
import random

# =========================
# KHỞI ĐỘNG
# =========================

pygame.init()
pygame.joystick.init()

if pygame.joystick.get_count() == 0:
    print("Không tìm thấy tay cầm!")
    pygame.quit()
    exit()

controller = pygame.joystick.Joystick(0)
controller.init()

print("Tay cầm:", controller.get_name())

# =========================
# CỬA SỔ
# =========================

WIDTH = 800
HEIGHT = 600

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Mouse & Cheese")

clock = pygame.time.Clock()

# =========================
# MÀU
# =========================

WHITE = (255, 255, 255)
BLUE = (50, 120, 255)
YELLOW = (255, 200, 0)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GRAY = (100, 100, 100)

# =========================
# CHUỘT
# =========================

mouse_x = WIDTH // 2
mouse_y = HEIGHT // 2

mouse_radius = 25
speed = 5

# =========================
# CHEESE
# =========================

cheese_x = random.randint(50, WIDTH - 50)
cheese_y = random.randint(50, HEIGHT - 50)

cheese_radius = 18

# =========================
# SCORE
# =========================

score = 0

# =========================
# MÁU
# =========================

lives = 3

# =========================
# MÈO
# =========================

cat_x = 0
cat_y = 0

cat_radius = 30
cat_speed = 4

cat_active = False
cat_warning = False

warning_timer = 0
attack_timer = 0

# Thời gian bất tử ngắn sau khi bị đánh
hit_cooldown = 0

# Thời gian trước khi mèo tiếp theo xuất hiện
cat_spawn_timer = 180

# =========================
# FONT
# =========================

font = pygame.font.Font(None, 36)
warning_font = pygame.font.Font(None, 48)
game_over_font = pygame.font.Font(None, 72)

# =========================
# GAME LOOP
# =========================

running = True

while running:

    # =========================
    # EVENTS
    # =========================

    for event in pygame.event.get():

        if event.type == pygame.QUIT:
            running = False

    # =========================
    # TIMER
    # =========================

    if hit_cooldown > 0:
        hit_cooldown -= 1

    if cat_spawn_timer > 0:
        cat_spawn_timer -= 1

    # =========================
    # ANALOG
    # =========================

    analog_x = controller.get_axis(0)
    analog_y = controller.get_axis(1)

    # Dead zone
    if abs(analog_x) < 0.15:
        analog_x = 0

    if abs(analog_y) < 0.15:
        analog_y = 0

    # =========================
    # D-PAD
    # =========================

    dpad_x = 0
    dpad_y = 0

    if controller.get_numhats() > 0:
        dpad_x, dpad_y = controller.get_hat(0)

    # =========================
    # ĐIỀU KHIỂN
    # =========================

    # Đây là mapping bạn xác nhận là đúng
    move_x = analog_x + dpad_x
    move_y = analog_y - dpad_y

    mouse_x += move_x * speed
    mouse_y += move_y * speed

    # =========================
    # ĐI QUA CẠNH ĐỐI DIỆN
    # =========================

    if mouse_x < -mouse_radius:
        mouse_x = WIDTH + mouse_radius

    elif mouse_x > WIDTH + mouse_radius:
        mouse_x = -mouse_radius

    if mouse_y < -mouse_radius:
        mouse_y = HEIGHT + mouse_radius

    elif mouse_y > HEIGHT + mouse_radius:
        mouse_y = -mouse_radius

    # =========================
    # ĂN CHEESE
    # =========================

    distance = (
        (mouse_x - cheese_x) ** 2
        + (mouse_y - cheese_y) ** 2
    ) ** 0.5

    if distance < mouse_radius + cheese_radius:

        score += 10

        cheese_x = random.randint(
            50,
            WIDTH - 50
        )

        cheese_y = random.randint(
            50,
            HEIGHT - 50
        )

        print("Score:", score)

    # =========================
    # MÈO BẮT ĐẦU WARNING
    # =========================

    if (
        not cat_active
        and not cat_warning
        and cat_spawn_timer <= 0
    ):

        cat_warning = True

        # Warning khoảng 1.5 giây
        warning_timer = 90

        # Chọn vị trí mèo xuất hiện
        side = random.choice([
            "left",
            "right",
            "top",
            "bottom"
        ])

        if side == "left":
            cat_x = -cat_radius
            cat_y = random.randint(
                cat_radius,
                HEIGHT - cat_radius
            )

        elif side == "right":
            cat_x = WIDTH + cat_radius
            cat_y = random.randint(
                cat_radius,
                HEIGHT - cat_radius
            )

        elif side == "top":
            cat_x = random.randint(
                cat_radius,
                WIDTH - cat_radius
            )
            cat_y = -cat_radius

        elif side == "bottom":
            cat_x = random.randint(
                cat_radius,
                WIDTH - cat_radius
            )
            cat_y = HEIGHT + cat_radius

    # =========================
    # WARNING COUNTDOWN
    # =========================

    if cat_warning:

        warning_timer -= 1

        if warning_timer <= 0:

            cat_warning = False
            cat_active = True

            # Mèo tấn công trong khoảng 2 giây
            attack_timer = 300

    # =========================
    # MÈO ĐUỔI CHUỘT
    # =========================

    if cat_active:

        dx = mouse_x - cat_x
        dy = mouse_y - cat_y

        distance_to_mouse = (
            dx ** 2 + dy ** 2
        ) ** 0.5

        if distance_to_mouse > 0:

            cat_x += (
                dx / distance_to_mouse
            ) * cat_speed

            cat_y += (
                dy / distance_to_mouse
            ) * cat_speed

        attack_timer -= 1

        if attack_timer <= 0:

            cat_active = False

            # Thời gian chờ trước mèo tiếp theo
            cat_spawn_timer = random.randint(
                120,
                300
            )

    # =========================
    # MÈO ĐỤNG CHUỘT
    # =========================

    if cat_active and hit_cooldown <= 0:

        distance_to_mouse = (
            (mouse_x - cat_x) ** 2
            + (mouse_y - cat_y) ** 2
        ) ** 0.5

        if distance_to_mouse < mouse_radius + cat_radius:

            lives -= 1

            print("Lives:", lives)

            # Mèo biến mất
            cat_active = False

            # Không cho bị hit liên tục
            hit_cooldown = 90

            # Mèo xuất hiện lại sau một khoảng thời gian
            cat_spawn_timer = random.randint(
                150,
                300
            )

    # =========================
    # VẼ
    # =========================

    screen.fill(WHITE)

    # -------------------------
    # CHEESE
    # -------------------------

    pygame.draw.circle(
        screen,
        YELLOW,
        (
            int(cheese_x),
            int(cheese_y)
        ),
        cheese_radius
    )

    # -------------------------
    # CHUỘT
    # -------------------------

    pygame.draw.circle(
        screen,
        BLUE,
        (
            int(mouse_x),
            int(mouse_y)
        ),
        mouse_radius
    )

    # -------------------------
    # MÈO
    # -------------------------

    if cat_active:

        pygame.draw.circle(
            screen,
            GRAY,
            (
                int(cat_x),
                int(cat_y)
            ),
            cat_radius
        )

    # -------------------------
    # WARNING
    # -------------------------

    if cat_warning:

        warning_text = warning_font.render(
            "WARNING! CAT ATTACK!",
            True,
            RED
        )

        screen.blit(
            warning_text,
            (
                WIDTH // 2
                - warning_text.get_width() // 2,
                70
            )
        )

    # -------------------------
    # SCORE
    # -------------------------

    score_text = font.render(
        "Score: " + str(score),
        True,
        BLACK
    )

    screen.blit(
        score_text,
        (20, 20)
    )

    # -------------------------
    # LIVES
    # -------------------------

    lives_text = font.render(
        "Lives: " + "♥ " * lives,
        True,
        RED
    )

    screen.blit(
        lives_text,
        (20, 55)
    )

    # =========================
    # GAME OVER
    # =========================

    if lives <= 0:

        game_over_text = game_over_font.render(
            "GAME OVER",
            True,
            RED
        )

        screen.blit(
            game_over_text,
            (
                WIDTH // 2
                - game_over_text.get_width() // 2,
                HEIGHT // 2
                - game_over_text.get_height() // 2
            )
        )

    # =========================
    # UPDATE
    # =========================

    pygame.display.flip()

    clock.tick(60)

# =========================
# THOÁT
# =========================

pygame.quit()
