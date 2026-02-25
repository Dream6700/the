import pygame
import random
import sys

# 初始化
pygame.init()

# 窗口
WIDTH, HEIGHT = 1000, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Mini Minecraft")

# 方块大小
BLOCK_SIZE = 40
GRAVITY = 0.5
JUMP_POWER = -10

# 颜色
SKY = (135, 206, 235)
DIRT = (120, 72, 0)
GRASS = (34, 177, 76)
STONE = (100, 100, 100)
PLAYER_COLOR = (255, 0, 0)

# 世界大小
WORLD_WIDTH = 200
WORLD_HEIGHT = 20

# 创建世界
world = [[0 for _ in range(WORLD_HEIGHT)] for _ in range(WORLD_WIDTH)]

# 生成地形
for x in range(WORLD_WIDTH):
    height = random.randint(8, 12)
    for y in range(height, WORLD_HEIGHT):
        if y == height:
            world[x][y] = 1  # 草
        elif y < height + 3:
            world[x][y] = 2  # 土
        else:
            world[x][y] = 3  # 石头

# 玩家
player = pygame.Rect(100, 100, 30, 50)
velocity_y = 0
on_ground = False
camera_x = 0

clock = pygame.time.Clock()

def draw_world():
    for x in range(WORLD_WIDTH):
        for y in range(WORLD_HEIGHT):
            block = world[x][y]
            if block != 0:
                rect = pygame.Rect(
                    x * BLOCK_SIZE - camera_x,
                    y * BLOCK_SIZE,
                    BLOCK_SIZE,
                    BLOCK_SIZE
                )
                if block == 1:
                    pygame.draw.rect(screen, GRASS, rect)
                elif block == 2:
                    pygame.draw.rect(screen, DIRT, rect)
                elif block == 3:
                    pygame.draw.rect(screen, STONE, rect)

def get_block_rects():
    rects = []
    for x in range(WORLD_WIDTH):
        for y in range(WORLD_HEIGHT):
            if world[x][y] != 0:
                rect = pygame.Rect(
                    x * BLOCK_SIZE,
                    y * BLOCK_SIZE,
                    BLOCK_SIZE,
                    BLOCK_SIZE
                )
                rects.append(rect)
    return rects

# 主循环
while True:
    clock.tick(60)
    screen.fill(SKY)

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        # 鼠标破坏/放置
        if event.type == pygame.MOUSEBUTTONDOWN:
            mx, my = pygame.mouse.get_pos()
            world_x = (mx + camera_x) // BLOCK_SIZE
            world_y = my // BLOCK_SIZE
            if 0 <= world_x < WORLD_WIDTH and 0 <= world_y < WORLD_HEIGHT:
                if event.button == 1:
                    world[world_x][world_y] = 0
                if event.button == 3:
                    world[world_x][world_y] = 2

    keys = pygame.key.get_pressed()

    dx = 0
    if keys[pygame.K_a]:
        dx = -5
    if keys[pygame.K_d]:
        dx = 5
    if keys[pygame.K_w] and on_ground:
        velocity_y = JUMP_POWER

    velocity_y += GRAVITY
    dy = velocity_y

    # X轴移动
    player.x += dx
    for block in get_block_rects():
        if player.colliderect(block):
            if dx > 0:
                player.right = block.left
            if dx < 0:
                player.left = block.right

    # Y轴移动
    player.y += dy
    on_ground = False
    for block in get_block_rects():
        if player.colliderect(block):
            if dy > 0:
                player.bottom = block.top
                velocity_y = 0
                on_ground = True
            if dy < 0:
                player.top = block.bottom
                velocity_y = 0

    # 摄像机
    camera_x = player.x - WIDTH // 2

    draw_world()
    pygame.draw.rect(screen, PLAYER_COLOR,
                     (player.x - camera_x, player.y, player.width, player.height))

    pygame.display.flip()
