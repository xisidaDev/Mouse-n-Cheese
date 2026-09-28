import pygame, random, sys, os

pygame.init()
pygame.joystick.init()

has_controller = pygame.joystick.get_count() > 0
if has_controller:
    controller = pygame.joystick.Joystick(0)
    controller.init()

WIDTH, HEIGHT = 900, 700
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Mouse & Cheese")
clock = pygame.time.Clock()

# ==========================================
# CẤU HÌNH KÍCH THƯỚC CHUỘT VÀ MÈO
# ==========================================
MOUSE_SIZE = (100, 100)   # Kích thước ảnh Chuột (Chiều rộng, Chiều cao)
CAT_SIZE = (100, 100)     # Kích thước ảnh Mèo (Chiều rộng, Chiều cao)

mouse_radius = MOUSE_SIZE[0] // 2
cat_radius = CAT_SIZE[0] // 2
cheese_radius = 18      # Bán kính phô mai hình tròn

# ==========================================
# NẠP VÀ CHỈNH KÍCH THƯỚC HÌNH ẢNH
# ==========================================
BASE_DIR = os.path.dirname(os.path.abspath(__file__))

# 1. Nạp ảnh Chuột
mouse_path = os.path.join(BASE_DIR, "Meru.png")
mouse_img = pygame.image.load(mouse_path)
mouse_img = pygame.transform.scale(mouse_img, MOUSE_SIZE)

# 2. Nạp ảnh Mèo (Thay "cat.png" nếu file ảnh của bạn tên khác)
cat_path = os.path.join(BASE_DIR, "Tatchumi.png")
cat_img = pygame.image.load(cat_path)
cat_img = pygame.transform.scale(cat_img, CAT_SIZE)

# Màu sắc & Biến game
WHITE, YELLOW, BLACK, RED = (255, 255, 255), (255, 200, 0), (0, 0, 0), (255, 0, 0)
mouse_x, mouse_y, speed = WIDTH // 2, HEIGHT // 2, 5
cheese_x, cheese_y = random.randint(50, WIDTH - 50), random.randint(50, HEIGHT - 50)
score, lives = 0, 3

cat_x, cat_y, cat_speed = 0, 0, 4
cat_active, cat_warning = False, False
warning_timer, attack_timer, hit_cooldown, cat_spawn_timer = 0, 0, 0, 180

font = pygame.font.Font(None, 36)
warning_font = pygame.font.Font(None, 48)
game_over_font = pygame.font.Font(None, 72)

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT: running = False

    if lives > 0:
        if hit_cooldown > 0: hit_cooldown -= 1
        if cat_spawn_timer > 0: cat_spawn_timer -= 1

        move_x, move_y = 0, 0
        if has_controller:
            ax, ay = controller.get_axis(0), controller.get_axis(1)
            ax = 0 if abs(ax) < 0.15 else ax
            ay = 0 if abs(ay) < 0.15 else ay
            hx, hy = controller.get_hat(0) if controller.get_numhats() > 0 else (0, 0)
            move_x, move_y = ax + hx, ay - hy
        else:
            keys = pygame.key.get_pressed()
            if keys[pygame.K_LEFT] or keys[pygame.K_a]: move_x = -1
            if keys[pygame.K_RIGHT] or keys[pygame.K_d]: move_x = 1
            if keys[pygame.K_UP] or keys[pygame.K_w]: move_y = -1
            if keys[pygame.K_DOWN] or keys[pygame.K_s]: move_y = 1

        mouse_x += move_x * speed
        mouse_y += move_y * speed

        if mouse_x < -mouse_radius: mouse_x = WIDTH + mouse_radius
        elif mouse_x > WIDTH + mouse_radius: mouse_x = -mouse_radius
        if mouse_y < -mouse_radius: mouse_y = HEIGHT + mouse_radius
        elif mouse_y > HEIGHT + mouse_radius: mouse_y = -mouse_radius

        if ((mouse_x - cheese_x)**2 + (mouse_y - cheese_y)**2)**0.5 < mouse_radius + cheese_radius:
            score += 10
            cheese_x, cheese_y = random.randint(50, WIDTH - 50), random.randint(50, HEIGHT - 50)

        if not cat_active and not cat_warning and cat_spawn_timer <= 0:
            cat_warning, warning_timer = True, 90
            side = random.choice(["left", "right", "top", "bottom"])
            if side == "left": cat_x, cat_y = -cat_radius, random.randint(cat_radius, HEIGHT - cat_radius)
            elif side == "right": cat_x, cat_y = WIDTH + cat_radius, random.randint(cat_radius, HEIGHT - cat_radius)
            elif side == "top": cat_x, cat_y = random.randint(cat_radius, WIDTH - cat_radius), -cat_radius
            elif side == "bottom": cat_x, cat_y = random.randint(cat_radius, WIDTH - cat_radius), HEIGHT + cat_radius

        if cat_warning:
            warning_timer -= 1
            if warning_timer <= 0: cat_warning, cat_active, attack_timer = False, True, 300

        if cat_active:
            dx, dy = mouse_x - cat_x, mouse_y - cat_y
            dist = (dx**2 + dy**2)**0.5
            if dist > 0: cat_x += (dx / dist) * cat_speed; cat_y += (dy / dist) * cat_speed
            attack_timer -= 1
            if attack_timer <= 0: cat_active, cat_spawn_timer = False, random.randint(120, 300)

        if cat_active and hit_cooldown <= 0:
            if ((mouse_x - cat_x)**2 + (mouse_y - cat_y)**2)**0.5 < mouse_radius + cat_radius:
                lives -= 1
                cat_active, hit_cooldown, cat_spawn_timer = False, 90, random.randint(150, 300)

    # Vẽ màn hình
    screen.fill(WHITE)

    # 1. Vẽ Phô mai (Vẫn là hình tròn vàng như cũ)
    pygame.draw.circle(screen, YELLOW, (int(cheese_x), int(cheese_y)), cheese_radius)

    # 2. Vẽ Chuột bằng ảnh
    screen.blit(mouse_img, (int(mouse_x - mouse_radius), int(mouse_y - mouse_radius)))

    # 3. Vẽ Mèo bằng ảnh
    if cat_active and lives > 0:
        screen.blit(cat_img, (int(cat_x - cat_radius), int(cat_y - cat_radius)))

    if cat_warning and lives > 0:
        w_txt = warning_font.render("KAZEHAYA TATSUMI IS COMING!", True, RED)
        screen.blit(w_txt, (WIDTH // 2 - w_txt.get_width() // 2, 110))

    s_txt = font.render("Score: " + str(score), True, BLACK)
    l_txt = font.render("Lives: " + "<3 " * lives, True, RED)
    screen.blit(s_txt, (20, 20))
    screen.blit(l_txt, (20, 55))

    if lives <= 0:
        go_txt = game_over_font.render("GAME OVER", True, RED)
        screen.blit(go_txt, (WIDTH // 2 - go_txt.get_width() // 2, HEIGHT // 2 - go_txt.get_height() // 2))

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
sys.exit()
# THOÁT
# =========================

pygame.quit()
