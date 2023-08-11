---
layout: post
title: Portfolio
subtitle: Picture Portfolio
cover-img: assets/img/aa.png
gh-repo: daattali/beautiful-jekyll
tags: [Portfolio]
comments: true
---

```python
import pygame
import random
import math

# 게임 초기화
pygame.init()
WINDOW_WIDTH = 800
WINDOW_HEIGHT = 600
window = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption("Tower Defense")
clock = pygame.time.Clock()
is_running = True
COIN_START = 200
PLAYER_START_HP = 100
BACKGROUND_COLOR = (0, 0, 0)
WHITE = (255, 255, 255)
FONT = pygame.font.Font(None, 36)

# 클래스 정의
class Tower(pygame.sprite.Sprite):
    def __init__(self, pos_x, pos_y):
        super().__init__()
        self.image = pygame.Surface((40, 40))
        self.image.fill((255, 0, 0))
        self.rect = self.image.get_rect()
        self.rect.centerx = pos_x
        self.rect.centery = pos_y
        self.attack_damage = 20
        self.attack_range = 150
        self.attack_speed = 1
        self.last_attack_time = 0

    def attack(self, target):
        now = pygame.time.get_ticks()
        if now - self.last_attack_time >= 1000 / self.attack_speed:
            bullet = Bullet(self.rect.centerx, self.rect.centery, target)
            all_sprites.add(bullet)
            bullets.add(bullet)
            self.last_attack_time = now

class Monster(pygame.sprite.Sprite):
    def __init__(self, pos_x, pos_y, hp, speed):
        super().__init__()
        self.image = pygame.Surface((30, 30))
        self.image.fill((0, 255, 0))
        self.rect = self.image.get_rect()
        self.rect.x = pos_x
        self.rect.y = pos_y
        self.hp = hp
        self.max_hp = hp
        self.speed = speed

    def update(self):
        self.rect.x += self.speed

class Bullet(pygame.sprite.Sprite):
    def __init__(self, pos_x, pos_y, target):
        super().__init__()
        self.image = pygame.Surface((10, 10))
        self.image.fill((255, 255, 0))
        self.rect = self.image.get_rect()
        self.rect.centerx = pos_x
        self.rect.centery = pos_y
        self.target = target
        self.speed = 5
        self.damage = 30

    def update(self):
        angle = math.atan2(self.target.rect.centery - self.rect.centery, self.target.rect.centerx - self.rect.centerx)
        self.rect.x += self.speed * math.cos(angle)
        self.rect.y += self.speed * math.sin(angle)
        if pygame.sprite.collide_rect(self, self.target):
            self.target.hp -= self.damage
            bullets.remove(self)
            all_sprites.remove(self)

class Player:
    def __init__(self):
        self.coin = COIN_START
        self.hp = PLAYER_START_HP

# 게임 객체 초기화
all_sprites = pygame.sprite.Group()
towers = pygame.sprite.Group()
monsters = pygame.sprite.Group()
bullets = pygame.sprite.Group()

player = Player()

# 웨이브 데이터
waves = [
    [{"hp": 50, "speed": 1}, {"hp": 80, "speed": 0.8}],
    [{"hp": 60, "speed": 1.2}, {"hp": 100, "speed": 1}]
]

wave_index = 0
wave_timer = pygame.time.get_ticks()

# 몬스터 스폰 변수
monster_spawn_interval = 2000
monster_spawn_timer = 0

# 추가 함수
def show_next_wave_prompt():
    font = pygame.font.Font(None, 36)
    text = font.render("다음 웨이브로 넘어가시겠습니까? (y/n)", True, WHITE)
    text_rect = text.get_rect(center=(WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2))
    window.blit(text, text_rect)
    pygame.display.flip()

    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                pos = pygame.mouse.get_pos()
                if text_rect.collidepoint(pos):
                    return True, False
                elif text_rect.collidepoint(pos):
                    return False, True

def increase_wave_difficulty(wave):
    for monster_data in wave:
        monster_data["hp"] += 10

def update_wave():
    global wave_index
    if wave_index < len(waves):
        wave = waves[wave_index]
        if show_next_wave_prompt():
            wave_index += 1
            if wave_index < len(waves):
                increase_wave_difficulty(wave)

# 게임 루프
while is_running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            is_running = False
        if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            pos = pygame.mouse.get_pos()
            if player.coin >= 50:
                tower = Tower(pos[0], pos[1])
                all_sprites.add(tower)
                towers.add(tower)
                player.coin -= 50

    now = pygame.time.get_ticks()
    if now - wave_timer >= 5000 and wave_index < len(waves):
        wave_timer = now
        wave = waves[wave_index]
        monster_spawn_timer = now

        for monster_data in wave:
            monster = Monster(0, WINDOW_HEIGHT // 2, monster_data["hp"], monster_data["speed"])
            all_sprites.add(monster)
            monsters.add(monster)
            monster_spawn_timer += monster_spawn_interval  # 몬스터 생성 간격 누적
        wave_index += 1
        if wave_index >= len(waves):
            wave_index = 0

    if now - monster_spawn_timer >= monster_spawn_interval and wave_index < len(waves):
        monster_data = waves[wave_index][0]
        monster = Monster(0, WINDOW_HEIGHT // 2, monster_data["hp"], monster_data["speed"])
        all_sprites.add(monster)
        monsters.add(monster)
        monster_spawn_timer += monster_spawn_interval  # 몬스터 생성 간격 누적

    for monster in monsters:
        monster.update()

    for tower in towers:
        target = None
        for monster in monsters:
            if abs(monster.rect.centery - tower.rect.centery) < tower.attack_range and monster.rect.x > tower.rect.x:
                if not target or monster.rect.x > target.rect.x:
                    target = monster
        if target:
            tower.attack(target)

    for bullet in bullets:
        bullet.update()

    for monster in monsters:
        if monster.rect.x >= WINDOW_WIDTH:
            player.hp -= 10
            monsters.remove(monster)
            all_sprites.remove(monster)
        if monster.hp <= 0:
            player.coin += 20
            monsters.remove(monster)
            all_sprites.remove(monster)

    window.fill(BACKGROUND_COLOR)
    all_sprites.draw(window)

    for monster in monsters:
        hp_text = FONT.render(f"HP: {monster.hp}/{monster.max_hp}", True, WHITE)
        window.blit(hp_text, (monster.rect.x, monster.rect.y - 20))

    hud_text = FONT.render(f"Coin: {player.coin}   HP: {player.hp}", True, WHITE)
    window.blit(hud_text, (20, 20))

    pygame.display.flip()
    clock.tick(60)

    if player.hp <= 0:
        is_running = False
    if len(monsters) == 0:
        next_wave, continue_game = show_next_wave_prompt()
        if next_wave:
            update_wave()
        elif continue_game:
            pass  # 게임 계속 진행

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 클래스 정의

### `Tower` 클래스

- 타워 객체를 나타내는 클래스임 /
- `__init__(self, pos_x, pos_y)`: 타워의 초기 위치(`pos_x`, `pos_y`) 및 속성을 설정해줌 /
- `attack(self, target)`: 지정된 타겟을 공격하는 메서드로, 탄환을 생성하여 타겟에게 공격을 가해줌 /

### `Monster` 클래스

- 몬스터 객체를 나타내는 클래스임 /
- `__init__(self, pos_x, pos_y, hp, speed)`: 몬스터의 초기 위치(`pos_x`, `pos_y`), 체력(`hp`), 속도(`speed`) 및 기타 속성을 설정해줌 /
- `update(self)`: 몬스터의 이동 동작을 업데이트하는 메서드임 /

### `Bullet` 클래스

- 탄환 객체를 나타내는 클래스임 /
- `__init__(self, pos_x, pos_y, target)`: 탄환의 초기 위치(`pos_x`, `pos_y`) 및 타겟(`target`)을 설정해줌 /
- `update(self)`: 탄환의 이동 동작 및 타겟과의 충돌 검사를 처리하는 메서드임 /

### `Player` 클래스

- 플레이어 정보를 나타내는 클래스임 /
- `__init__(self)`: 초기 코인과 체력을 설정하여 플레이어 객체를 초기화해줌 /

## 주요 변수

### 게임 창 및 기본 설정

- `WINDOW_WIDTH`: 게임 창 가로 크기
- `WINDOW_HEIGHT`: 게임 창 세로 크기
- `window`: Pygame을 이용한 게임 창 객체
- `is_running`: 게임 실행 여부
- `COIN_START`: 게임 시작 시 초기 코인
- `PLAYER_START_HP`: 게임 시작 시 초기 플레이어 체력
- `BACKGROUND_COLOR`: 게임 창 배경색
- `WHITE`: 텍스트 색상
- `FONT`: 게임 내 폰트 객체

### 게임 객체 관리

- `all_sprites`: Pygame의 `Group` 객체로, 모든 게임 객체 관리
- `towers`: Pygame의 `Group` 객체로, 타워 객체 관리
- `monsters`: Pygame의 `Group` 객체로, 몬스터 객체 관리
- `bullets`: Pygame의 `Group` 객체로, 탄환 객체 관리

### 플레이어 정보

- `player`: `Player` 클래스의 인스턴스로, 플레이어 정보 관리
- `player / coin`: 플레이어의 코인 정보
- `player / hp`: 플레이어의 체력 정보

### 웨이브 및 몬스터 스폰

- `waves`: 게임 웨이브 정보를 담은 리스트
- `wave_index`: 현재 진행 중인 웨이브의 인덱스
- `wave_timer`: 웨이브 간 경과 시간 측정
- `monster_spawn_interval`: 몬스터 생성 간격 설정
- `monster_spawn_timer`: 몬스터 생성 시간 간격 관리
