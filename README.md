# Shooter-Game
Videojuego de Nave
import pygame, random
 
Width = 800
Height = 600
Black = (0, 0, 0)
White = (255, 255, 255)
GREEN = (0 , 255, 0)

pygame.init()
pygame.mixer.init()
screen = pygame.display.set_mode((Width, Height))
pygame.display.set_caption("Shooter")
clock = pygame.time.Clock()

#Dibujo del Marcador
def draw_text(surface, text, size, x, y):
    font = pygame.font.SysFont("serif", size) 
    text_surface = font.render(text,True,White)
    text_rect = text_surface.get_rect()
    text_rect.midtop = (x, y)
    surface.blit(text_surface, text_rect)

#Dibujo de la Barra de salud
def draw_shield_bar(surface, x, y, percentage):
    BAR_LENGHT = 100
    BAR_HEIGHT = 10
    fill = (percentage/100)*BAR_LENGHT
    border = pygame.Rect(x,y,BAR_LENGHT,BAR_HEIGHT)
    fill = pygame.Rect(x,y,fill,BAR_HEIGHT)
    pygame.draw.rect(surface, GREEN, fill)
    pygame.draw.rect(surface, White, border, 2)


# Definicion del jugador
class Player(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = pygame.image.load("Imagen de juego/player2.png").convert()
        self.image.set_colorkey(Black)
        self.rect = self.image.get_rect()
        self.rect.centerx = Width // 2
        self.rect.bottom = Height - 10
        self.speed_x = 0
        #Escudo
        self.shield = 100
# Movimiento del jugador
    def update(self):
        self.speed_x = 0
        keystate = pygame.key.get_pressed()
        if keystate[pygame.K_LEFT]:
            self.speed_x = -5
        if keystate[pygame.K_RIGHT]:
            self.speed_x = 5
        self.rect.x += self.speed_x
        if self.rect.right > Width:
            self.rect.right = Width
        if self.rect.left < 0:
            self.rect.left = 0

    def shoot(self):
        bullet = Bullet(self.rect.centerx, self.rect.top)
        all_sprites.add(bullet)
        bullets.add(bullet)
        laser_sound.play()

# Definicion del enemigo
class Meteor(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = random.choice(meteor_images)  
        self.image.set_colorkey(Black)
        self.rect = self.image.get_rect()
        self.rect.x = random.randrange(Width - self.rect.width)
        self.rect.y = random.randrange(-140, -100)
        self.speedy = random.randrange(1, 10)
        self.speedx = random.randrange(-5, 5)

    def update(self):
        self.rect.y += self.speedy
        self.rect.x += self.speedx
        if self.rect.top > Height + 10 or self.rect.left < -40 or self.rect.right > Width + 40:
            self.rect.x = random.randrange(Width - self.rect.width)
            self.rect.y = random.randrange(-100, -40)
            self.speedy = random.randrange(1, 10)

# Definicion del disparo del jugador
class Bullet(pygame.sprite.Sprite):
    def __init__(self,x,y):
        super().__init__()
        self.image = pygame.image.load("Imagen de juego/laser1.png")
        self.image.set_colorkey(Black)
        self.rect = self.image.get_rect()
        self.rect.y = y
        self.rect.centerx = x
        self.speedy = -10

    def update(self):
        self.rect.y += self.speedy
        if self.rect.bottom < 0:
            self.kill()

# Definicion de la animacion de explosiones del enemigo
class Explosion(pygame.sprite.Sprite):
    def __init__(self, center):
        super().__init__()
        self.image = explosion_anim[0]
        self.rect = self.image.get_rect()
        self.rect.center = center
        self.frame = 0
        self.last_update = pygame.time.get_ticks()
        self.frame_rate = 50

    def update(self) -> None:
        now = pygame.time.get_ticks()
        if now - self.last_update > self.frame_rate:
            self.last_update = now
            self.frame += 1
            if self.frame == len(explosion_anim):
                self.kill()
            else:
                center = self.rect.center
                self.image = explosion_anim[self.frame]
                self.rect = self.image.get_rect()
                self.rect.center = center

def show_go_screen():
    screen.blit(background, [0,0])
    draw_text(screen, "SHOOTER", 65, Width // 2, Height // 4)
    draw_text(screen, "Instrucciones van aqui" , 27, Width // 2, Height // 2)
    draw_text(screen, "Press Key", 20, Width // 2, Height * 3/4)
    pygame.display.flip()
    waiting = True
    while waiting:
        clock.tick(60)
        for event in  pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
            if event.type == pygame.KEYUP:
                waiting = False

    
# Imagenes de Meteoros
meteor_images = []
meteor_list = ["Imagen de juego/meteorGrey_big1.png","Imagen de juego/meteorGrey_big2.png","Imagen de juego/meteorGrey_big3.png","Imagen de juego/meteorGrey_big4.png",
               "Imagen de juego/meteorGrey_med1.png","Imagen de juego/meteorGrey_med2.png", "Imagen de juego/meteorGrey_small1.png", "Imagen de juego/meteorGrey_small2.png",
               "Imagen de juego/meteorGrey_tiny1.png", "Imagen de juego/meteorGrey_tiny2.png"]

for img in meteor_list:
    meteor_images.append(pygame.image.load(img).convert())

#Explosion Imagenes
explosion_anim = []
for i in range(9):
    file = "Imagen de juego/regularExplosion0{}.png".format(i)
    img = pygame.image.load(file).convert()
    img.set_colorkey(Black)
    img_scale = pygame.transform.scale(img, (70,70))
    explosion_anim.append(img_scale)


# Cargar imagen de fondo
background = pygame.image.load("Imagen de juego/background.png").convert()

# Cargar sonidos
laser_sound = pygame.mixer.Sound("Sonido de juego/Laser2.ogg")
explosion_sound = pygame.mixer.Sound("Sonido de juego/explosion.wav")
pygame.mixer.music.load("Sonido de juego/music.ogg")
pygame.mixer.music.set_volume(0.2)    

all_sprites = pygame.sprite.Group()
meteor_list = pygame.sprite.Group()
bullets = pygame.sprite.Group()

# Variable de Marcador
pygame.mixer.music.play(loops=-1)

#Game over
game_over = True


running = True
while running:
    if game_over:

        show_go_screen()

        game_over = False
        all_sprites = pygame.sprite.Group()
        meteor_list = pygame.sprite.Group()
        bullets = pygame.sprite.Group()
        player = Player()
        all_sprites.add(player)
        for i in range(8):
            meteor = Meteor()
            all_sprites.add(meteor)
            meteor_list.add(meteor)
            
        score = 0

    clock.tick(60)
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                player.shoot()
    
    all_sprites.update()
    #colisiones - meteoro - laser
    hits = pygame.sprite.groupcollide(meteor_list, bullets, True, True)
    for hit in hits:
        score += 10
        explosion_sound.play()
        explosion = Explosion(hit.rect.center)
        all_sprites.add(explosion)
        meteor = Meteor()
        all_sprites.add(meteor)
        meteor_list.add(meteor)

    #Chequiar colisiones
    hits = pygame.sprite.spritecollide(player, meteor_list, True)
    for hit in hits:
        player.shield -= 25
        meteor = Meteor()
        all_sprites.add(meteor)
        meteor_list.add(meteor)
        if player.shield <= 0:
            game_over = True

    screen.blit(background, [0, 0])

    all_sprites.draw(screen)

    #Marcador
    draw_text(screen, str(score), 25, Width // 2, 10)

    #Escudo
    draw_shield_bar(screen, 5, 5, player.shield)

    pygame.display.flip()
pygame.quit() 

