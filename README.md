from pygame import *
from random import randint
from time import time as timer

#фоновая музыка
mixer.init()
mixer.music.load('space.ogg')
mixer.music.play()
fire_sound = mixer.Sound('fire.ogg')

#шрифты и надписи
font.init()
font1 = font.SysFont('Arial', 80)
font2 = font.SysFont('Arial', 36)
winn = font1.render('YOU WIN!', True, (255,255,255))
losee = font1.render('YOU LOSE!', True, (180,0,0))

# нам нужны такие картинки:
img_back = "galaxy.jpg"
img_hero = "rocket.png" 
img_enemy = "ufo.png"
img_bullet = "bullet.png"

score = 0 # сбито кораблей
lost = 0 # пропущено кораблей
lose = "negr.jpg"
win = "leon.jpg"
ast = "pelmen.jpg"
# класс-родитель для других спрайтов
class GameSprite(sprite.Sprite):
  # конструктор класса
    def __init__(self, player_image, player_x, player_y, size_x, size_y, player_speed):
        # Вызываем конструктор класса (Sprite):
        sprite.Sprite.__init__(self)

        # каждый спрайт должен хранить свойство image - изображение
        self.image = transform.scale(image.load(player_image), (size_x, size_y))
        self.speed = player_speed

        # каждый спрайт должен хранить свойство rect - прямоугольник, в который он вписан
        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y
 
  # метод, отрисовывающий героя на окне
    def reset(self):
        window.blit(self.image, (self.rect.x, self.rect.y))

# класс главного игрока
class Player(GameSprite):
    # метод для управления спрайтом стрелками клавиатуры
    def update(self):
        keys = key.get_pressed()
        if keys[K_LEFT] and self.rect.x > 5:
            self.rect.x -= self.speed
        if keys[K_RIGHT] and self.rect.x < win_width - 80:
            self.rect.x += self.speed
        if keys[K_UP] and self.rect.y > 5:
            self.rect.y -= self.speed
        if keys[K_DOWN] and self.rect.y < 400:
            self.rect.y += self.speed
  # метод "выстрел" (используем место игрока, чтобы создать там пулю)
    def fire(self):
        bullet = Bullet(img_bullet,self.rect.centerx,self.rect.top,15,20,-15)
        bullets.add(bullet)
        
class Bullet(GameSprite): 
    def update(self): 
        self.rect.y += self.speed
        if self.rect.y < 0:
            self.kill()
            
lost = 0  
score = 0
goal = 25
max_lost = 10
life = 3
class Enemy(GameSprite):
    # движение врага
    def update(self):
        self.rect.y += self.speed
        global lost
        # исчезает, если дойдет до края экрана
        if self.rect.y > win_height:
            self.rect.x = randint(80, win_width - 80)
            self.rect.y = 0
            lost= lost + 1

win_width = 700
win_height = 500
display.set_caption("Shooter")
window = display.set_mode((win_width, win_height))
background = transform.scale(image.load(img_back), (win_width, win_height))
pelms = sprite.Group()
for i in range(1, 3):
    pe1men = Enemy(ast, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
    pelms.add(pe1men)
ship = Player(img_hero, 5, win_height - 100, 80, 100, 10)
monsters = sprite.Group()
for i in range(1, 10):
    monster = Enemy(img_enemy, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
    monsters.add(monster)

bullets = sprite.Group()
finish = False
run = True
while run:
    # событие нажатия на кнопку Закрыть
    for e in event.get():
        if e.type == QUIT:
            run = False
        elif e.type == KEYDOWN:
            if e.key == K_SPACE:
                fire_sound.play()
                ship.fire()

    if not finish:
        window.blit(background,(0,0))
        text = font2.render("Счет: " + str(score), 1, (0, 255, 150))
        window.blit(text, (10, 20))
        text_lose = font2.render("Пропущено: " + str(lost), 1, (255, 150, 0))
        window.blit(text_lose, (10, 50))
        if life == 3:
            life_color = (0, 150, 0)
        if life == 2:
            life_color = (150, 150, 0)
        if life == 1:
            life_color = (150, 0, 0)
        text_life = font1.render(str(life),1,life_color)
        window.blit(text_life, (670, 0))
        collides = sprite.groupcollide(monsters,bullets,True,True)
        for c in collides:
            score+=1
            monster = Enemy(img_enemy, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
            monsters.add(monster)
        if sprite.spritecollide(ship,monsters,False) or sprite.spritecollide(ship,pelms,False):
            sprite.spritecollide(ship,monsters,True)
            sprite.spritecollide(ship,pelms,True)
            life = life - 1 
        if life == 0 or lost >= max_lost:
            finish = True
            window.blit(losee, (200,200))
        if score >= goal:
            finish = True
            window.blit(winn, (200,200))
        

        ship.update()
        monsters.update()
        pelms.update()
        bullets.update()
        ship.reset()
        pelms.draw(window)
        monsters.draw(window)
        bullets.draw(window)
        display.update()
    else:
        finish = False
        score = 0
        lost = 0
        life = 3
        for b in bullets:
            b.kill()
        for m in monsters:
            m.kill()
        for a in pelms:
            a.kill()
        time.delay(3000)
        for i in range(1, 10): 
            monster = Enemy(img_enemy, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
            monsters.add(monster)
        for i in range(1, 3):
            pe1men = Enemy(ast, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
            pelms.add(pe1men)


    time.delay(50)
