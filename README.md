# Palette-Zone-version-3-
import pygame
import sys
import math
import random
import os
from pygame import gfxdraw

# Initialize pygame
pygame.init()
pygame.mixer.init()

# Screen dimensions
SCREEN_WIDTH = 1200
SCREEN_HEIGHT = 800
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Palette Zone for Autistic")

# Colors - Pastel palette with requested purple/pink buttons
BACKGROUND = (230, 240, 255)
PANEL_BG = (200, 230, 255)
BUTTON_BG = (180, 220, 240)  # Light blue
BUTTON_HOVER = (160, 200, 230)  # Slightly darker blue
TEXT_COLOR = (40, 40, 80)
ACCENT_COLOR = (255, 150, 150)
GRID_COLOR = (180, 200, 230)
HIGHLIGHT_COLOR = (255, 240, 100)
PURPLE_BUTTON = (180, 120, 220)  # Purple button
LIGHT_PINK = (255, 182, 193)    # Light pink button

# Pastel color palette
COLORS = [
    (255, 204, 204), (255, 229, 204), (255, 255, 204), (204, 255, 204),
    (204, 255, 255), (204, 204, 255), (255, 204, 255), (230, 230, 250),
    (255, 218, 185), (175, 238, 238), (240, 255, 240), (255, 228, 225),
    (220, 220, 255), (255, 222, 173), (173, 216, 230), (240, 248, 255),
    (255, 245, 238), (245, 255, 250), (255, 239, 213), (224, 255, 255)
]

# Drawing constants
BRUSH_SIZES = [5, 10, 20, 30]
BRUSH_SHAPES = ["round", "square", "marker"]

class DrawingApp:
    def __init__(self):
        self.clock = pygame.time.Clock()
        self.font = pygame.font.SysFont("comicsansms", 36)
        self.small_font = pygame.font.SysFont("comicsansms", 28)
        self.title_font = pygame.font.SysFont("comicsansms", 48, bold=True)
        
        # Drawing state
        self.current_color = COLORS[0]
        self.brush_size = BRUSH_SIZES[1]
        self.brush_shape = BRUSH_SHAPES[0]
        self.drawing = False
        self.last_pos = None
        
        # Book and page state
        self.books = ["Animals", "Fruits", "Vegetables"]
        self.current_book_index = 0
        self.current_page = 1
        self.total_pages = 30
        self.page_type = "coloring"  # coloring, differences, words
        self.section = 1  # Each section has 6 pages
        
        # Drawing surfaces
        self.reference_surface = pygame.Surface((SCREEN_WIDTH//2 - 20, SCREEN_HEIGHT - 250))
        self.reference_surface.fill((255, 255, 255))
        self.drawing_surface = pygame.Surface((SCREEN_WIDTH//2 - 20, SCREEN_HEIGHT - 250))
        self.drawing_surface.fill((255, 255, 255))
        
        # Load sample images
        self.load_sample_images()
        
        # Sound
        self.sound_enabled = True
        self.load_sounds()
        
        # UI elements
        self.buttons = self.create_buttons()
        self.color_buttons = self.create_color_buttons()
        self.brush_buttons = self.create_brush_buttons()
        self.download_buttons = self.create_download_buttons()
        
        # Reward system
        self.reward_active = False
        self.reward_timer = 0
        self.reward_type = ""
        
        # License information
        self.license_info = {
            "owner": "Your Name",
            "email": "your@email.com",
            "license": "Open Educational Resource - Free for non-commercial use"
        }
        
        # Words similarity data
        self.words_data = [
            {"word": "Apple", "image": "fruit", "options": ["Fruit", "Animal", "Color", "Shape"]},
            {"word": "Lion", "image": "animal", "options": ["Animal", "Fruit", "Place", "Action"]},
            {"word": "Circle", "image": "shape", "options": ["Shape", "Color", "Number", "Food"]},
            {"word": "Red", "image": "color", "options": ["Color", "Emotion", "Animal", "Size"]},
            {"word": "Carrot", "image": "vegetable", "options": ["Vegetable", "Fruit", "Color", "Shape"]},
            {"word": "Elephant", "image": "animal", "options": ["Animal", "Place", "Thing", "Action"]},
            {"word": "Banana", "image": "fruit", "options": ["Fruit", "Vegetable", "Color", "Shape"]},
            {"word": "Broccoli", "image": "vegetable", "options": ["Vegetable", "Fruit", "Tree", "Animal"]}
        ]
        self.current_word_index = 0
        self.selected_option = None
        
        # Differences data
        self.differences = []
        self.found_differences = []
        self.current_difference_set = 0
        
        # Book content data
        self.book_content = {
            "Animals": ["Lion", "Elephant", "Giraffe", "Monkey", "Zebra", "Tiger"],
            "Fruits": ["Apple", "Banana", "Orange", "Grapes", "Strawberry", "Watermelon"],
            "Vegetables": ["Carrot", "Broccoli", "Tomato", "Cucumber", "Potato", "Lettuce"]
        }
        
        # Download counters
        self.downloads = 0
        
    def load_sample_images(self):
        """Create sample images for coloring and reference"""
        # Create a simple reference image
        self.reference_surface.fill((255, 255, 255))
        self.draw_sample_image(self.reference_surface, colored=True)
        
        # Create a numbered outline for coloring
        self.drawing_surface.fill((255, 255, 255))
        self.draw_sample_image(self.drawing_surface, colored=False)
    
    def draw_sample_image(self, surface, colored=False):
        """Draw a sample image based on current book and page"""
        width, height = surface.get_size()
        center_x, center_y = width // 2, height // 2
        
        current_book = self.books[self.current_book_index]
        book_items = self.book_content[current_book]
        item_index = (self.current_page - 1) % len(book_items)
        current_item = book_items[item_index]
        
        # Draw different images based on book type
        if current_book == "Animals":
            self.draw_animal(surface, center_x, center_y, current_item, colored)
        elif current_book == "Fruits":
            self.draw_fruit(surface, center_x, center_y, current_item, colored)
        else:  # Vegetables
            self.draw_vegetable(surface, center_x, center_y, current_item, colored)
    
    def draw_animal(self, surface, cx, cy, animal, colored):
        """Draw an animal based on the name"""
        if animal == "Lion":
            # Draw lion body
            pygame.draw.ellipse(surface, (180, 150, 100) if not colored else (200, 170, 50), 
                              (cx - 80, cy - 50, 160, 120))
            # Draw head
            pygame.draw.circle(surface, (180, 150, 100) if not colored else (220, 190, 80), 
                             (cx, cy - 80), 50)
            # Draw mane
            for i in range(12):
                angle = i * math.pi/6
                px = cx + math.cos(angle) * 60
                py = cy - 80 + math.sin(angle) * 60
                pygame.draw.circle(surface, (150, 120, 80) if not colored else (180, 150, 50), 
                                 (px, py), 20)
            # Draw eyes
            pygame.draw.circle(surface, (40, 40, 40), (cx - 20, cy - 90), 8)
            pygame.draw.circle(surface, (40, 40, 40), (cx + 20, cy - 90), 8)
            # Draw nose
            pygame.draw.circle(surface, (100, 60, 60), (cx, cy - 70), 10)
            # Draw mouth
            pygame.draw.arc(surface, (100, 60, 60), (cx - 20, cy - 70, 40, 30), 0, math.pi, 3)
            
        elif animal == "Elephant":
            # Draw body
            pygame.draw.ellipse(surface, (180, 180, 180) if not colored else (200, 200, 200), 
                              (cx - 100, cy - 60, 200, 120))
            # Draw head
            pygame.draw.circle(surface, (180, 180, 180) if not colored else (200, 200, 200), 
                             (cx + 80, cy - 30), 50)
            # Draw trunk
            trunk_points = [
                (cx + 80, cy - 30),
                (cx + 120, cy - 20),
                (cx + 130, cy),
                (cx + 120, cy + 20),
                (cx + 100, cy + 10)
            ]
            pygame.draw.polygon(surface, (180, 180, 180) if not colored else (200, 200, 200), trunk_points)
            # Draw ears
            pygame.draw.ellipse(surface, (180, 180, 180) if not colored else (180, 180, 220), 
                              (cx - 50, cy - 100, 80, 120))
            pygame.draw.ellipse(surface, (180, 180, 180) if not colored else (180, 180, 220), 
                              (cx + 30, cy - 100, 80, 120))
            # Draw eyes
            pygame.draw.circle(surface, (40, 40, 40), (cx + 70, cy - 40), 8)
            # Draw legs
            for i in range(4):
                pygame.draw.rect(surface, (180, 180, 180) if not colored else (200, 200, 200), 
                               (cx - 80 + i*50, cy + 50, 30, 50))
        
        # Add more animals as needed...
    
    def draw_fruit(self, surface, cx, cy, fruit, colored):
        """Draw a fruit based on the name"""
        if fruit == "Apple":
            # Draw apple
            pygame.draw.circle(surface, (150, 150, 150) if not colored else (220, 60, 60), 
                             (cx, cy), 60)
            # Draw stem
            pygame.draw.rect(surface, (100, 80, 60), (cx - 5, cy - 70, 10, 20))
            # Draw leaf
            leaf_points = [
                (cx + 5, cy - 70),
                (cx + 40, cy - 90),
                (cx + 30, cy - 70)
            ]
            pygame.draw.polygon(surface, (150, 150, 150) if not colored else (100, 180, 100), leaf_points)
        
        elif fruit == "Banana":
            # Draw banana
            banana_points = [
                (cx - 70, cy - 20),
                (cx - 40, cy - 50),
                (cx + 40, cy - 30),
                (cx + 20, cy + 10),
                (cx - 50, cy + 20)
            ]
            pygame.draw.polygon(surface, (150, 150, 150) if not colored else (240, 220, 80), banana_points)
        
        # Add more fruits as needed...
    
    def draw_vegetable(self, surface, cx, cy, vegetable, colored):
        """Draw a vegetable based on the name"""
        if vegetable == "Carrot":
            # Draw carrot body
            pygame.draw.ellipse(surface, (150, 150, 150) if not colored else (255, 140, 0), 
                              (cx - 30, cy - 80, 60, 160))
            # Draw leaves
            for i in range(5):
                angle = math.pi/2 + i * math.pi/3
                start_x = cx
                start_y = cy - 80
                end_x = cx + math.cos(angle) * 50
                end_y = cy - 80 + math.sin(angle) * 50
                pygame.draw.line(surface, (150, 150, 150) if not colored else (100, 180, 100), 
                               (start_x, start_y), (end_x, end_y), 8)
        
        elif vegetable == "Broccoli":
            # Draw stem
            pygame.draw.rect(surface, (150, 150, 150) if not colored else (100, 180, 100), 
                           (cx - 20, cy, 40, 100))
            # Draw florets
            for i in range(20):
                floret_x = cx + random.randint(-50, 50)
                floret_y = cy - random.randint(20, 100)
                size = random.randint(15, 30)
                pygame.draw.circle(surface, (150, 150, 150) if not colored else (60, 150, 60), 
                                 (floret_x, floret_y), size)
        
        # Add more vegetables as needed...
    
    def load_sounds(self):
        """Load or create nature sounds"""
        try:
            # In a real implementation, you would load actual sound files
            # For this example, we'll create placeholder sounds
            self.sounds = {
                "birds": pygame.mixer.Sound(pygame.sndarray.array(bytearray([128]*44100))),
                "water": pygame.mixer.Sound(pygame.sndarray.array(bytearray([128]*44100)))
            }
            self.sounds["birds"].set_volume(0.3)
            self.sounds["water"].set_volume(0.2)
        except:
            self.sounds = {}
            self.sound_enabled = False
    
    def create_buttons(self):
        """Create navigation and control buttons"""
        buttons = []
        
        # Navigation buttons
        buttons.append({
            "rect": pygame.Rect(50, SCREEN_HEIGHT - 80, 150, 50),
            "text": "Previous",
            "action": "prev_page"
        })
        
        buttons.append({
            "rect": pygame.Rect(SCREEN_WIDTH - 200, SCREEN_HEIGHT - 80, 150, 50),
            "text": "Next",
            "action": "next_page"
        })
        
        buttons.append({
            "rect": pygame.Rect(SCREEN_WIDTH//2 - 75, SCREEN_HEIGHT - 80, 150, 50),
            "text": "Check Work",
            "action": "check_work"
        })
        
        # Book selection buttons
        for i, book in enumerate(self.books):
            buttons.append({
                "rect": pygame.Rect(50 + i * 200, 20, 180, 50),
                "text": book,
                "action": "select_book",
                "book_index": i
            })
        
        # For words page
        buttons.append({
            "rect": pygame.Rect(SCREEN_WIDTH//2 - 100, SCREEN_HEIGHT - 150, 200, 50),
            "text": "Submit Answer",
            "action": "submit_answer",
            "visible": False
        })
        
        return buttons
    
    def create_download_buttons(self):
        """Create download buttons as requested"""
        buttons = []
        
        # Download Free button (purple with light pink text)
        buttons.append({
            "rect": pygame.Rect(SCREEN_WIDTH - 450, SCREEN_HEIGHT - 80, 180, 50),
            "text": "Download Free",
            "action": "download_image",
            "color": PURPLE_BUTTON,
            "text_color": LIGHT_PINK
        })
        
        # Save as HQ button
        buttons.append({
            "rect": pygame.Rect(SCREEN_WIDTH - 250, SCREEN_HEIGHT - 80, 200, 50),
            "text": "Save as HQ Image",
            "action": "download_hq",
            "color": PURPLE_BUTTON,
            "text_color": LIGHT_PINK
        })
        
        return buttons
    
    def create_color_buttons(self):
        """Create buttons for the color palette"""
        buttons = []
        button_size = 40
        margin = 10
        start_x = 50
        start_y = 100
        
        for i, color in enumerate(COLORS):
            row = i // 5
            col = i % 5
            x = start_x + col * (button_size + margin)
            y = start_y + row * (button_size + margin)
            
            buttons.append({
                "rect": pygame.Rect(x, y, button_size, button_size),
                "color": color,
                "action": "select_color"
            })
        
        return buttons
    
    def create_brush_buttons(self):
        """Create buttons for brush sizes"""
        buttons = []
        start_x = 200
        start_y = 100
        size = 40
        
        for i, brush_size in enumerate(BRUSH_SIZES):
            buttons.append({
                "rect": pygame.Rect(start_x + i * (size + 10), start_y, size, size),
                "size": brush_size,
                "action": "select_brush"
            })
        
        return buttons
    
    def determine_page_type(self):
        """Determine the current page type based on position in section"""
        position_in_section = (self.current_page - 1) % 6
        
        if position_in_section < 2:
            return "coloring"
        elif position_in_section < 4:
            return "differences"
        else:
            return "words"
    
    def update_page(self):
        """Update the page content based on current page number"""
        self.page_type = self.determine_page_type()
        self.section = (self.current_page - 1) // 6 + 1
        
        # Clear surfaces
        self.reference_surface.fill((255, 255, 255))
        self.drawing_surface.fill((255, 255, 255))
        
        if self.page_type == "coloring":
            self.draw_sample_image(self.reference_surface, colored=True)
            self.draw_sample_image(self.drawing_surface, colored=False)
        elif self.page_type == "differences":
            self.draw_difference_images()
        elif self.page_type == "words":
            self.draw_words_page()
    
    def draw_difference_images(self):
        """Draw images for the differences page with 15 differences"""
        # Create two similar images with differences
        width, height = self.reference_surface.get_size()
        center_x, center_y = width // 2, height // 2
        
        # Draw base image on both surfaces
        for surface in [self.reference_surface, self.drawing_surface]:
            # Draw background
            pygame.draw.rect(surface, (220, 240, 255), (0, 0, width, height))
            
            # Draw sun
            pygame.draw.circle(surface, (255, 240, 100), (100, 100), 50)
            
            # Draw house
            pygame.draw.rect(surface, (200, 180, 150), (center_x - 100, center_y - 50, 200, 150))
            pygame.draw.polygon(surface, (180, 80, 80), [
                (center_x - 120, center_y - 50),
                (center_x, center_y - 120),
                (center_x + 120, center_y - 50)
            ])
            
            # Draw door
            pygame.draw.rect(surface, (150, 100, 80), (center_x - 30, center_y + 50, 60, 100))
            pygame.draw.circle(surface, (100, 80, 60), (center_x + 20, center_y + 100), 5)
            
            # Draw windows
            pygame.draw.rect(surface, (180, 220, 255), (center_x - 80, center_y, 40, 40))
            pygame.draw.rect(surface, (180, 220, 255), (center_x + 40, center_y, 40, 40))
            
            # Draw chimney
            pygame.draw.rect(surface, (150, 100, 100), (center_x + 70, center_y - 120, 30, 40))
            
            # Draw tree
            pygame.draw.rect(surface, (120, 80, 50), (center_x - 180, center_y + 50, 30, 100))
            pygame.draw.circle(surface, (60, 150, 60), (center_x - 165, center_y + 30), 50)
            
            # Draw flowers
            for i in range(3):
                self.draw_flower(surface, center_x - 150 + i*50, center_y + 150, 10)
        
        # Add 15 differences to the drawing surface
        # 1. Extra cloud
        pygame.draw.circle(self.drawing_surface, (250, 250, 250), (center_x + 150, 80), 30)
        pygame.draw.circle(self.drawing_surface, (250, 250, 250), (center_x + 170, 70), 25)
        pygame.draw.circle(self.drawing_surface, (250, 250, 250), (center_x + 130, 70), 20)
        
        # 2. Missing window
        pygame.draw.rect(self.drawing_surface, (200, 180, 150), (center_x + 40, center_y, 40, 40))
        
        # 3. Different door color
        pygame.draw.rect(self.drawing_surface, (180, 140, 100), (center_x - 30, center_y + 50, 60, 100))
        
        # 4. Extra flower
        self.draw_flower(self.drawing_surface, center_x - 150, center_y + 150, 15)
        
        # 5. Different sun rays
        for i in range(8):
            angle = i * math.pi/4
            start_x = 100 + math.cos(angle) * 55
            start_y = 100 + math.sin(angle) * 55
            end_x = 100 + math.cos(angle) * 75
            end_y = 100 + math.sin(angle) * 75
            pygame.draw.line(self.drawing_surface, (255, 240, 100), 
                           (start_x, start_y), (end_x, end_y), 4)
        
        # 6. Missing chimney
        pygame.draw.rect(self.drawing_surface, (180, 80, 80), (center_x + 70, center_y - 120, 30, 40))
        
        # 7. Different tree position
        pygame.draw.rect(self.drawing_surface, (120, 80, 50), (center_x - 150, center_y + 50, 30, 100))
        pygame.draw.circle(self.drawing_surface, (60, 150, 60), (center_x - 135, center_y + 30), 50)
        
        # 8. Different flower colors
        for i in range(3):
            pygame.draw.circle(self.drawing_surface, (255, 150, 180), 
                             (center_x - 150 + i*50, center_y + 150), 15)
        
        # 9. Extra bird
        self.draw_bird(self.drawing_surface, center_x + 100, 150, 15)
        
        # 10. Missing bush
        pygame.draw.rect(self.drawing_surface, (220, 240, 255), (center_x + 100, center_y + 120, 80, 30))
        
        # 11. Different roof color
        pygame.draw.polygon(self.drawing_surface, (160, 60, 60), [
            (center_x - 120, center_y - 50),
            (center_x, center_y - 120),
            (center_x + 120, center_y - 50)
        ])
        
        # 12. Extra window
        pygame.draw.rect(self.drawing_surface, (180, 220, 255), (center_x, center_y - 30, 40, 40))
        
        # 13. Different ground texture
        for i in range(10):
            x = random.randint(50, width - 50)
            y = random.randint(center_y + 150, height - 20)
            pygame.draw.circle(self.drawing_surface, (140, 200, 140), (x, y), 3)
        
        # 14. Missing fence
        for i in range(5):
            pygame.draw.rect(self.drawing_surface, (220, 240, 255), 
                          (50 + i*50, center_y + 150, 20, 40))
        
        # 15. Extra tree
        pygame.draw.rect(self.drawing_surface, (120, 80, 50), (center_x + 150, center_y + 50, 30, 100))
        pygame.draw.circle(self.drawing_surface, (60, 150, 60), (center_x + 165, center_y + 30), 50)
        
        # Store difference locations
        self.differences = [
            {"rect": pygame.Rect(center_x + 120, 50, 70, 40), "found": False},  # Cloud
            {"rect": pygame.Rect(center_x + 40, center_y, 40, 40), "found": False},  # Missing window
            {"rect": pygame.Rect(center_x - 30, center_y + 50, 60, 100), "found": False},  # Door color
            {"rect": pygame.Rect(center_x - 160, center_y + 140, 30, 30), "found": False},  # Flower
            {"rect": pygame.Rect(80, 80, 40, 40), "found": False},  # Sun rays
            {"rect": pygame.Rect(center_x + 70, center_y - 120, 30, 40), "found": False},  # Chimney
            {"rect": pygame.Rect(center_x - 160, center_y + 30, 100, 120), "found": False},  # Tree position
            {"rect": pygame.Rect(center_x - 160, center_y + 140, 150, 30), "found": False},  # Flower colors
            {"rect": pygame.Rect(center_x + 85, 135, 30, 30), "found": False},  # Bird
            {"rect": pygame.Rect(center_x + 100, center_y + 120, 80, 30), "found": False},  # Bush
            {"rect": pygame.Rect(center_x - 120, center_y - 120, 240, 70), "found": False},  # Roof color
            {"rect": pygame.Rect(center_x, center_y - 30, 40, 40), "found": False},  # Extra window
            {"rect": pygame.Rect(50, center_y + 150, width-100, height-center_y-150), "found": False},  # Ground texture
            {"rect": pygame.Rect(50, center_y + 150, width-100, 40), "found": False},  # Fence
            {"rect": pygame.Rect(center_x + 135, center_y + 30, 60, 120), "found": False}  # Extra tree
        ]
        self.found_differences = []
    
    def draw_words_page(self):
        """Draw the word similarity page"""
        # Draw background
        width, height = self.drawing_surface.get_size()
        self.drawing_surface.fill((220, 240, 255))
        
        # Get current word data
        word_data = self.words_data[self.current_word_index]
        
        # Draw word
        word_text = self.title_font.render(word_data["word"], True, TEXT_COLOR)
        self.drawing_surface.blit(word_text, (width//2 - word_text.get_width()//2, 50))
        
        # Draw options
        option_height = 60
        option_spacing = 20
        start_y = 200
        
        for i, option in enumerate(word_data["options"]):
            option_rect = pygame.Rect(150, start_y + i*(option_height + option_spacing), 
                                     width - 300, option_height)
            
            # Draw option box
            color = HIGHLIGHT_COLOR if self.selected_option == i else BUTTON_BG
            pygame.draw.rect(self.drawing_surface, color, option_rect, border_radius=15)
            pygame.draw.rect(self.drawing_surface, TEXT_COLOR, option_rect, 2, border_radius=15)
            
            # Draw option text
            option_text = self.font.render(option, True, TEXT_COLOR)
            self.drawing_surface.blit(option_text, 
                                    (option_rect.centerx - option_text.get_width()//2,
                                     option_rect.centery - option_text.get_height()//2))
    
    def handle_events(self):
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                return False
                
            if event.type == pygame.MOUSEBUTTONDOWN:
                if event.button == 1:  # Left mouse button
                    self.handle_click(event.pos)
                    
                    # Start drawing if on drawing surface and in coloring mode
                    if self.page_type == "coloring":
                        drawing_area = pygame.Rect(SCREEN_WIDTH//2 + 10, 150, 
                                                 SCREEN_WIDTH//2 - 20, SCREEN_HEIGHT - 250)
                        if drawing_area.collidepoint(event.pos):
                            self.drawing = True
                            self.last_pos = event.pos
                    
                    # Check for differences in differences mode
                    elif self.page_type == "differences":
                        drawing_area = pygame.Rect(SCREEN_WIDTH//2 + 10, 150, 
                                                 SCREEN_WIDTH//2 - 20, SCREEN_HEIGHT - 250)
                        if drawing_area.collidepoint(event.pos):
                            # Convert to surface coordinates
                            surface_pos = (event.pos[0] - (SCREEN_WIDTH//2 + 10), 
                                         event.pos[1] - 150)
                            
                            # Check if click is on a difference
                            for diff in self.differences:
                                if not diff["found"] and diff["rect"].collidepoint(surface_pos):
                                    diff["found"] = True
                                    self.found_differences.append(diff)
                                    self.give_reward()
                                    break
            
            if event.type == pygame.MOUSEBUTTONUP:
                if event.button == 1:
                    self.drawing = False
                    self.last_pos = None
            
            if event.type == pygame.MOUSEMOTION:
                if self.drawing and self.page_type == "coloring":
                    self.draw_on_surface(event.pos)
        
        return True
    
    def handle_click(self, pos):
        # Check color buttons
        for button in self.color_buttons:
            if button["rect"].collidepoint(pos):
                self.current_color = button["color"]
        
        # Check brush buttons
        for button in self.brush_buttons:
            if button["rect"].collidepoint(pos):
                self.brush_size = button["size"]
        
        # Check action buttons
        for button in self.buttons:
            if button["rect"].collidepoint(pos):
                if button["action"] == "prev_page":
                    self.current_page = max(1, self.current_page - 1)
                    self.update_page()
                elif button["action"] == "next_page":
                    self.current_page = min(self.total_pages, self.current_page + 1)
                    self.update_page()
                elif button["action"] == "check_work":
                    self.give_reward()
                elif button["action"] == "select_book":
                    self.current_book_index = button["book_index"]
                    self.current_page = 1
                    self.update_page()
                elif button["action"] == "submit_answer":
                    self.check_answer()
        
        # Check download buttons
        for button in self.download_buttons:
            if button["rect"].collidepoint(pos):
                if button["action"] == "download_image":
                    self.save_current_page("download")
                elif button["action"] == "download_hq":
                    self.save_current_page("hq")
        
        # Check word options
        if self.page_type == "words":
            width, height = self.drawing_surface.get_size()
            start_y = 200
            option_height = 60
            option_spacing = 20
            
            for i in range(len(self.words_data[self.current_word_index]["options"])):
                option_rect = pygame.Rect(SCREEN_WIDTH//2 + 10 + 150, 
                                        150 + start_y + i*(option_height + option_spacing),
                                        width - 300, option_height)
                
                if option_rect.collidepoint(pos):
                    self.selected_option = i
    
    def draw_on_surface(self, pos):
        """Draw on the drawing surface"""
        drawing_area = pygame.Rect(SCREEN_WIDTH//2 + 10, 150, 
                                 SCREEN_WIDTH//2 - 20, SCREEN_HEIGHT - 250)
        
        if drawing_area.collidepoint(pos):
            # Convert to surface coordinates
            surface_pos = (pos[0] - (SCREEN_WIDTH//2 + 10), pos[1] - 150)
            
            if self.last_pos:
                # Convert last position
                last_surface_pos = (self.last_pos[0] - (SCREEN_WIDTH//2 + 10), 
                                  self.last_pos[1] - 150)
                
                # Draw based on brush shape
                if self.brush_shape == "round":
                    pygame.draw.line(self.drawing_surface, self.current_color, 
                                   last_surface_pos, surface_pos, self.brush_size)
                elif self.brush_shape == "square":
                    pygame.draw.line(self.drawing_surface, self.current_color, 
                                   last_surface_pos, surface_pos, self.brush_size)
                else:  # Marker
                    pygame.draw.line(self.drawing_surface, self.current_color, 
                                   last_surface_pos, surface_pos, self.brush_size)
            
            # Draw endpoint
            if self.brush_shape == "round":
                pygame.draw.circle(self.drawing_surface, self.current_color, 
                                 surface_pos, self.brush_size // 2)
            elif self.brush_shape == "square":
                rect = pygame.Rect(surface_pos[0] - self.brush_size//2, 
                                 surface_pos[1] - self.brush_size//2,
                                 self.brush_size, self.brush_size)
                pygame.draw.rect(self.drawing_surface, self.current_color, rect)
            else:  # Marker
                pygame.draw.circle(self.drawing_surface, self.current_color, 
                                 surface_pos, self.brush_size // 2)
            
            self.last_pos = pos
    
    def give_reward(self):
        """Show a reward animation"""
        self.reward_active = True
        self.reward_timer = 180  # 3 seconds at 60 FPS
        self.reward_type = random.choice(["star", "flower", "clap"])
        
        # Play a reward sound if available
        if self.sound_enabled:
            pygame.mixer.Sound.play(random.choice(list(self.sounds.values())))
    
    def check_answer(self):
        """Check if the selected word option is correct"""
        if self.selected_option is None:
            return
            
        word_data = self.words_data[self.current_word_index]
        correct = word_data["options"][self.selected_option] == word_data["image"]
        
        if correct:
            self.give_reward()
            # Move to next word
            self.current_word_index = (self.current_word_index + 1) % len(self.words_data)
            self.selected_option = None
            self.draw_words_page()
        else:
            # Indicate incorrect answer
            self.reward_active = True
            self.reward_timer = 60
            self.reward_type = "incorrect"
    
    def save_current_page(self, quality):
        """Save the current page as an image"""
        try:
            # Create downloads directory if it doesn't exist
            if not os.path.exists("downloads"):
                os.makedirs("downloads")
            
            # Determine what to save
            current_book = self.books[self.current_book_index]
            
            if self.page_type == "coloring":
                # Save the drawing surface
                pygame.image.save(self.drawing_surface, 
                                 f"downloads/{current_book}_page_{self.current_page}.png")
            elif self.page_type == "differences":
                # Save the differences surface
                pygame.image.save(self.drawing_surface, 
                                 f"downloads/{current_book}_differences_{self.current_page}.png")
            else:  # words
                # Save the word surface
                pygame.image.save(self.drawing_surface, 
                                 f"downloads/{current_book}_words_{self.current_page}.png")
            
            self.downloads += 1
            self.give_reward()
            
        except Exception as e:
            print(f"Error saving image: {e}")
    
    def draw_reward(self):
        """Draw the reward animation"""
        if not self.reward_active:
            return
            
        center_x, center_y = SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2
        size = min(200, 200 * (1 - self.reward_timer / 180))
        
        if self.reward_type == "star":
            # Draw a star
            points = []
            for i in range(5):
                angle = math.pi/2 + i * 2*math.pi/5
                points.append((
                    center_x + size * math.cos(angle),
                    center_y + size * math.sin(angle))
                )
                points.append((
                    center_x + size
