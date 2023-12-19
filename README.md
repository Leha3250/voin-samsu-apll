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
win = font1.render('YOU WIN!',True,(255,255,255))
lose = font1.render('ПРОИГРЫШ',True,(180,5,5))
# герой
# враг
# сбито кораблей
# пропущено кораблей
points = 0
lost = 0
# класс-родитель для других спрайтов
class GameSprite(sprite.Sprite):
    # конструктор класса
    def __init__(self, player_image, player_x, player_y, size_x, size_y, player_speed):
        # Вызываем конструктор класса (Sprite):
        sprite.Sprite.__init__(self)
        # каждый спрайт должен хранить свойство image - изображение
        self.image = transform.scale(image.load(player_image), (size_x, size_y))
        self.size_x = size_x
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
    # метод "выстрел" (используем место игрока, чтобы создать там пулю)
    def fire(self):
        bullet = Bullet('bullet.png', self.rect.centerx-7, self.rect.top, 16, 20, -22)
        bullets.add(bullet)
        if self.rect.y <= 0:
            self.kill()
            
# класс пуль
class Bullet(GameSprite):
    def update(self):
        self.rect.y += self.speed



# класс спрайта-врага
class Enemy(GameSprite):
    def update(self):
        global lost
    # движение врага
        self.rect.y += self.speed
        # исчезает, если дойдет до края экрана
        if self.rect.y > win_height:
            self.rect.x = randint(80, win_width - 80)
            self.rect.y = 0
            if self.size_x == 80:
                lost += 1
# Создаем окошко
win_width = 700
win_height = 500
display.set_caption("Космическое вторжение")
window = display.set_mode((win_width, win_height))
background = transform.scale(image.load("galaxy.jpg"), (win_width, win_height))
# создаем спрайты
samsung = Player("rocket.png", win_width/2-40, win_height - 110, 80, 100, 10)
iphones = sprite.Group()
bullets = sprite.Group()
nokias = sprite.Group()
for i in range(5):
    iphone = Enemy("ufo.png",randint(0,win_width-80),-50,80,50,randint(1,5))
    iphones.add(iphone)
for i in range(5):
    nokia = Enemy("asteroid.png",randint(80,win_width-80),-40,50,50,randint(5, 15))
    nokias.add(nokia)
# переменная "игра закончилась": как только там True, в основном цикле перестают работать спрайты
finish = False
num_fire = 0
rel_time = False
# Основной цикл игры:
run = True # флаг сбрасывается кнопкой закрытия окна
while run:
    # событие нажатия на кнопку Закрыть
    for e in event.get():
        if e.type == QUIT:
            run = False
        elif e.type == KEYDOWN:
            if e.key == K_w:
                if num_fire < 5 and rel_time == False:
                    samsung.fire()
                    fire_sound.play()
                    num_fire += 1
                if num_fire >= 5 and rel_time == False:
                    last_time = timer()
                    rel_time = True
                
 
    if not finish:
        # обновляем фон
        window.blit(background,(0,0))
        # пишем текст на экране
        # производим движения спрайтов
        samsung.update()
        iphones.update()
        nokias.update()
        bullets.update()
        # обновляем их в новом местоположении при каждой итерации цикла
        samsung.reset()
        iphones.draw(window)
        bullets.draw(window)
        nokias.draw(window)
        #gthtpfhzlrf
        if rel_time == True:
            now_time = timer()

            if now_time - last_time < 3:
                reload = font2.render('Подождите... Телефон зарежается...',1,(255,0,0))
                window.blit(reload, (145, win_height-50))
            else:
                num_fire = 0
                rel_time = False

        text_point = font2.render('Счёт: '+str(points),True,(255,255,255))
        window.blit(text_point,(10,20))
 
        text_lost = font2.render('Пропущено: '+str(lost),True,(255,255,255))
        window.blit(text_lost,(10,60))

        sprites_list = sprite.groupcollide(iphones, bullets, True, True)
        for _ in sprites_list:
            points += 1
            iphone = Enemy("ufo.png",randint(0,win_width-80),-50,80,50,randint(1,5))
            iphones.add(iphone)

        if points >9:
            finish = True
            window.blit(win,(250,250))

        if lost > 2:
            finish = True
            window.blit(lose ,(200,250))

        display.update()
    else:
        time.delay(3000)
        finish = False
        points = 0
        lost = 0
        for b in bullets:
            b.kill()
        for i in iphones:
            i.kill() 

        for i in range(5):
            iphone = Enemy("ufo.png",randint(0,win_width-80),-50,80,50,randint(1,5))
            iphones.add(iphone)

    # цикл срабатывает каждую 0.05 секунд
    time.delay(50)
