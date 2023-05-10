import pygame
import random

from pygame import draw
from pygame.color import Color
from pygame.sprite import Sprite
from pygame.surface import Surface

from animation import Animation

pygame.init()

# 게임 화면 설정
image_list = ['background1.png','backgrounds1.png', ] #배경회면 늘려주기
size = (1000, 986)
screen = pygame.display.set_mode(size, pygame.FULLSCREEN)
pygame.display.set_caption("슈팅 게임")

#배경화면 클래스
class Background(Sprite):
    def __init__(self):
        self.i = 0
        self.sprite_image = image_list[self.i]
        self.image = pygame.image.load(self.sprite_image).convert()
        self.rect = self.image.get_rect()
        self.rect.y = 0
        self.dy = 1
        self.count = 0
        
        Sprite.__init__(self)

    def update(self):
        self.rect.y -= self.dy
        if self.rect.y == -400:
            self.rect.y = 0
            self.count += 1
            
        if self.count == 5:
            self.i = self.i + 1
            self.sprite_image = image_list[self.i]
            self.image = pygame.image.load(self.sprite_image).convert()
            self.rect = self.image.get_rect()
            self.count = 0
            
# 플레이어 클래스
class Player(Animation):
    def __init__(self):
        self.sprite_image = 'player1.png'
        self.sprite_width = 75
        self.sprite_height = 42
        self.sprite_columns = 10
        self.fps = 32
        self.speed = 5
        self.score = 10
        self.init_animation()

    def update(self):
        self.image.set_colorkey(Color(255, 255, 255))
        
        # 키 입력 처리
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            self.rect.x -= self.speed
        elif keys[pygame.K_RIGHT]:
            self.rect.x += self.speed
        if keys[pygame.K_UP]:
            self.rect.y -= self.speed
        elif keys[pygame.K_DOWN]:
            self.rect.y += self.speed
        
        # 화면을 벗어나지 않도록 위치 조정
        if self.rect.x < 0:
            self.rect.x = 0
        elif self.rect.x > 950:
            self.rect.x = 950
        if self.rect.y < 0:
            self.rect.y = 0
        elif self.rect.y > 986:
            self.rect.y = 986

# 적 클래스
class Enemy(Animation):
    def __init__(self, x, y):
        self.sprite_image = 'enemy1.png'
        self.sprite_width = 116
        self.sprite_height = 116
        self.sprite_columns = 10
        self.fps = 8
        self.speed = random.randint(1, 5) #스피드 랜덤
        self.init_animation()
        self.rect.x = x
        self.rect.y = y
        
    def update(self):
        self.image.set_colorkey(Color(255, 255, 255))
        
        self.rect.y += self.speed
        if self.rect.top > 1000:
            self.rect.y = 0

# 총알 클래스
class Bullet(Animation):
    def __init__(self):
        self.sprite_image = 'bullet.png'
        self.sprite_width = 8
        self.sprite_height = 8
        self.sprite_columns = 4
        self.speed = -10
        self.fps = 20
        self.init_animation()

    def update(self):
        self.rect.y += self.speed
        
    def shoot(self, x, y):
        self.rect.y = y
        self.rect.x = x

#보스의 클래스
class Boss(Animation):
    def __init__(self):
        self.sprite_image = 'Boss.png'
        self.sprite_width = 490
        self.sprite_height = 317
        self.sprite_columns = 0
        self.speed = -10
        self.fps = 32
        self.init_animation()

    def update(self):
        self.image.set_colorkey(Color(255, 255, 255))

        if self.rect.top > 350:
            self.rect.top = 350

class Item(Animation):
    def __init__(self, x, y):
        self.sprite_image = 'randombox1.png'
        self.sprite_width = 84
        self.sprite_height = 67
        self.sprite_columns = 0
        self.fps = 24
        self.speed = random.randint(1, 3) #스피드 랜덤
        self.init_animation()
        self.rect.x = x
        self.rect.y = y
        
    def update(self):
        self.image.set_colorkey(Color(255, 255, 255))

        self.rect.y += self.speed
        if self.rect.top > 1000:
            self.kill()

class Shield(Animation):
    def __init__(self):
        self.sprite_image = 'shield1.png'
        self.sprite_width = 92
        self.sprite_height = 55
        self.sprite_columns = 0
        self.fps = 8
        self.init_animation()

    def update(self):
        self.calc_next_frame()
        if self.current_frame == self.sprite_columns:
            self.current = 0

        rect = (self.sprite_width*self.current_frame, 0,
                self.sprite_width, self.sprite_height)

        self.image.blit(self.sprite_sheet, (0, 0), rect)
        self.image.set_colorkey(Color(255, 255, 255))
        
#이미지 그룹
background = Background()
background_group = pygame.sprite.Group()
background_group.add(background)

player = Player()
player.rect.x = 450
player.rect.y = 900
player_group = pygame.sprite.Group()
player_group.add(player)

boss = Boss()
boss.rect.x = 500
boss.rect.y = -5000
boss_group = pygame.sprite.Group()
boss_group.add(boss)

shield = Shield()
shield.rect.x = -5000
shield.rect.y = -5000
shield_group = pygame.sprite.Group()
shield_group.add(shield)

#변수
bullets = 0
enemy_count = 0
boss_count = 0
shields = 0
item_li = 0
count = 0
space_bar = 0
shield_score = 0
boss_score = 0
blt_count = 0

#클래스 리스트
cls_list = [shield, plus]

bullet_group = pygame.sprite.Group()

for i in range(1000):
    b = Bullet()
    b.rect.x = -1000
    b.rect.y = -1000
    bullet_group.add(b)

#적의 좌표 생성
enemys_x = []
enemys_y = []

for i in range(10):
    enemys_x.append(random.randint(0, 884))
    enemys_y.append(random.randint(-400, 0))

enemy_group = pygame.sprite.Group()

for k in range(10):
    enemy = Enemy(enemys_x[k], enemys_y[k])
    enemy_group.add(enemy)

#아이템의 좌표 생성
items_x = []
items_y = []

for j in range(10):
    items_x.append(random.randint(0, 950))
    items_y.append(random.randint(-400, 0))

item_group = pygame.sprite.Group()

# 게임 루프
running = True
clock = pygame.time.Clock()

while running:
    # 이벤트 처리
    for event in pygame.event.get(): 
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                #중심에 들어가게 만들기
                space_bar += 1
                bullet_group.sprites()[bullets].shoot(player.rect.x + 67 / 2, player.rect.y)
                bullets += 1
                if bullets == 1000:
                    running = False

    #스코어 점수가 400이 되면 보스 출현
    if player.score % 400 == 0:
        boss.rect.x = 250
        boss.rect.y = 100
        boss_group.add(boss)

    #스코어가 200씩 늘어날 때 마다 쏜다.
    if player.score % 200 == 0:
        items = Item(items_x[item_li], items_y[item_li])
        item_group.add(items)
        item_li += 1
        player.score += 10

    if item_li == 10:
        item_li = 0

    # 게임 상태 업데이트
    player_group.update()
    bullet_group.update()
    enemy_group.update()
    boss_group.update()
    item_group.update()
    shield_group.update()

    # 충돌 처리
    if enemy.alive():
        hits = pygame.sprite.groupcollide(enemy_group, bullet_group, True, True)
        if hits:
            player.score += 10
            t = list(hits.keys())[0]
            t.rect.y = -200
            enemy_group.add(t)

    if boss.alive():
        hitss = pygame.sprite.groupcollide(boss_group, bullet_group, False, True)
        if hitss:
            boss_count += 1
            if boss_count == 30:
                boss_count = 0
                boss.kill()
                boss_score += 1

    if boss_score == 1:
        boss.rect.y = -5000
        boss_group.add(boss)
        boss_score = 0

    if player.alive():
        hit = pygame.sprite.groupcollide(item_group, player_group, True, False)
        if hit:
            shields += 1
            shield.rect.x = (player.rect.x + player.rect.width/2) - shield.rect.width/2
            shield.rect.y = player.rect.y
            count += 1
            shield_score += 1
    
    if shield_score == 1:
        shield_group.add(shield)
        shield_score = 0

    if count == 1:
        shield.rect.x = (player.rect.x + player.rect.width/2) - shield.rect.width/2
        shield.rect.y = player.rect.y
            
    if shield.alive():
        pygame.sprite.groupcollide(shield_group, enemy_group, True, True)
            
    if player.alive():
        hitse = pygame.sprite.groupcollide(player_group, enemy_group, True, True)
        if hitse:
            running = False

    # 화면 그리기
    background_group.update()
    background_group.draw(screen)
    player_group.draw(screen)
    bullet_group.draw(screen)
    enemy_group.draw(screen)
    boss_group.draw(screen)
    item_group.draw(screen)
    shield_group.draw(screen)

    # 점수 출력
    font = pygame.font.Font(None, 30)
    score_text = font.render('Score: {}'.format(player.score), True, (0, 0, 0))
    screen.blit(score_text, (10, 10))

    font = pygame.font.Font(None, 30)
    score_text = font.render('boss: {}'.format(boss_count), True, (0, 0, 0))
    screen.blit(score_text, (10, 30))

    # 화면 업데이트
    pygame.display.update()

    # 프레임 설정
    clock.tick(60)

pygame.quit()
