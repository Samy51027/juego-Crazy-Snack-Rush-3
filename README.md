# juego-Crazy-Snack-Rush-3
import pygame
import sys

# --- CONFIGURACIÓN INICIAL ---
pygame.init()
ANCHO, ALTO = 800, 600
pantalla = pygame.display.set_mode((ANCHO, ALTO))
pygame.display.set_caption("Rio Koi - Crazy Snack Rush TEC")
reloj = pygame.time.Clock()

# Colores Temáticos Rio Koi
NEGRO_FONDO = (15, 15, 20)
ROJO_KOI = (200, 40, 40)
ORO_FAROLA = (255, 214, 102)
BLANCO = (255, 255, 255)
GRIS_COCINA = (50, 50, 55)

# --- CLASES DEL PROYECTO ---

class Chef:
    """Clase que representa a los amigos cocineros."""
    def __init__(self, nombre, x, y, color):
        self.nombre = nombre
        self.puntos = 0
        self.rect = pygame.Rect(x, y, 40, 40) 
        self.color = color
        self.item_cargado = None 
        self.esta_activo = False
        self.velocidad = 5

    def mover(self, teclas):
        """Maneja el movimiento con el teclado."""
        if self.esta_activo:
            if teclas[pygame.K_LEFT] and self.rect.left > 0:
                self.rect.x -= self.velocidad
            if teclas[pygame.K_RIGHT] and self.rect.right < ANCHO:
                self.rect.x += self.velocidad
            if teclas[pygame.K_UP] and self.rect.top > 0:
                self.rect.y -= self.velocidad
            if teclas[pygame.K_DOWN] and self.rect.bottom < ALTO:
                self.rect.y += self.velocidad

    def dibujar(self, superficie):
        # Cuerpo del chef
        pygame.draw.rect(superficie, self.color, self.rect, border_radius=5)
        # Gorro de chef (detalle visual)
        pygame.draw.rect(superficie, BLANCO, (self.rect.x + 5, self.rect.y - 10, 30, 15), border_radius=3)
        # Indicador de selección
        if self.esta_activo:
            pygame.draw.polygon(superficie, ORO_FAROLA, [
                (self.rect.centerx, self.rect.top - 20),
                (self.rect.centerx - 5, self.rect.top - 30),
                (self.rect.centerx + 5, self.rect.top - 30)
            ])

class Estacion:
    """Clase base para las estaciones de trabajo."""
    def __init__(self, nombre, x, y, ancho, alto, color, tipo):
        self.nombre = nombre
        self.rect = pygame.Rect(x, y, ancho, alto)
        self.color = color
        self.tipo = tipo # 'despensa', 'preparacion' o 'entrega'

    def dibujar(self, superficie):
        pygame.draw.rect(superficie, self.color, self.rect, border_radius=3)
        pygame.draw.rect(superficie, (200, 200, 200), self.rect, 2, border_radius=3)
        
        fuente = pygame.font.SysFont("Arial", 14)
        txt = fuente.render(self.nombre, True, (255, 255, 255))
        superficie.blit(txt, (self.rect.x, self.rect.y - 18))

class Cocina:
    def __init__(self):
        self.tiempo = 180 
        self.chef1 = Chef("Amigo A", 300, 300, (200, 80, 80))
        self.chef2 = Chef("Amigo B", 450, 300, (80, 80, 200))
        self.chef1.esta_activo = True 
        self.lista_chefs = [self.chef1, self.chef2] 
        
        self.estaciones = [
            # Despensas (Basadas en ingredientes del documento)
            Estacion("Arroz", 0, 100, 60, 80, (240, 240, 240), "despensa"),
            Estacion("Pescado", 0, 200, 60, 80, (255, 100, 100), "despensa"),
            Estacion("Alga", 0, 300, 60, 80, (30, 60, 30), "despensa"),
            
            # Estaciones de Trabajo (Basadas en fuentes )
            Estacion("Tabla Picar", 200, 0, 100, 60, (150, 110, 80), "preparacion"), 
            Estacion("Sartén/Wok", 350, 0, 100, 60, (80, 80, 80), "preparacion"), 
            Estacion("Freidora", 500, 0, 100, 60, (200, 150, 50), "preparacion"), 
            
            # Estación de Entrega [cite: 32]
            Estacion("ENTREGA", 740, 200, 60, 150, (0, 150, 0), "entrega") 
        ]
        
    def cambiar_chef(self):
        self.chef1.esta_activo = not self.chef1.esta_activo
        self.chef2.esta_activo = not self.chef2.esta_activo

    def dibujar_escenario(self, superficie):
        superficie.fill(GRIS_COCINA)
        for est in self.estaciones:
            est.dibujar(superficie)

# --- FUNCIONES DE INTERFAZ ---

def mostrar_pantalla_inicio():
    fuente_titulo = pygame.font.SysFont("Verdana", 80, bold=True)
    fuente_sub = pygame.font.SysFont("Verdana", 25)
    
    esperando = True
    while esperando:
        pantalla.fill(NEGRO_FONDO)
        img_titulo = fuente_titulo.render("RIO KOI", True, ROJO_KOI)
        img_sub = fuente_sub.render("Presiona ENTER para comenzar", True, BLANCO)
        
        pantalla.blit(img_titulo, (ANCHO//2 - img_titulo.get_width()//2, 200))
        pantalla.blit(img_sub, (ANCHO//2 - img_sub.get_width()//2, 350))
        
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_RETURN:
                    esperando = False
        pygame.display.flip()
        reloj.tick(30)

# --- BUCLE PRINCIPAL ---

def juego():
    mi_cocina = Cocina()
    corriendo = True
    
    while corriendo:
        teclas = pygame.key.get_pressed()
        
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                corriendo = False
            
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_TAB:
                    mi_cocina.cambiar_chef()

        # Actualización
        for chef in mi_cocina.lista_chefs:
            chef.mover(teclas)

        # Dibujo
        mi_cocina.dibujar_escenario(pantalla)
        for chef in mi_cocina.lista_chefs:
            chef.dibujar(pantalla)
        
        # UI
        fuente = pygame.font.SysFont("Arial", 20)
        txt = fuente.render("TAB: Cambiar Chef | Flechas: Mover", True, BLANCO)
        pantalla.blit(txt, (20, ALTO - 40))

        pygame.display.flip()
        reloj.tick(60)

if __name__ == "__main__":
    mostrar_pantalla_inicio()
    juego()
    pygame.quit()
