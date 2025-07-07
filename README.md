# Add these missing methods to the DrawingApp class

def draw_flower(self, surface, x, y, size):
    """Draw a simple flower"""
    # Stem
    pygame.draw.line(surface, (100, 180, 100), (x, y), (x, y + size*2), 3)
    # Center
    pygame.draw.circle(surface, (255, 220, 60), (x, y), size//2)
    # Petals
    for angle in [0, math.pi/2, math.pi, 3*math.pi/2]:
        px = x + math.cos(angle) * size
        py = y + math.sin(angle) * size
        pygame.draw.circle(surface, (255, 150, 180), (px, py), size//2)

def draw_bird(self, surface, x, y, size):
    """Draw a simple bird"""
    # Body
    pygame.draw.ellipse(surface, (120, 120, 180), (x - size, y - size//2, size*2, size))
    # Head
    pygame.draw.circle(surface, (120, 120, 180), (x + size, y), size//2)
    # Beak
    pygame.draw.polygon(surface, (240, 180, 60), [
        (x + size*1.5, y),
        (x + size*2, y),
        (x + size*1.5, y + size//3)
    ])
    # Eye
    pygame.draw.circle(surface, (255, 255, 255), (x + size*1.2, y - size//4), size//6)
    pygame.draw.circle(surface, (0, 0, 0), (x + size*1.2, y - size//4), size//12)
    # Wing
    wing_points = [
        (x, y),
        (x - size//2, y - size),
        (x + size//2, y)
    ]
    pygame.draw.polygon(surface, (100, 100, 150), wing_points)

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
            # Outer points
            angle = math.pi/2 + i * 2*math.pi/5
            points.append((
                center_x + size * math.cos(angle),
                center_y + size * math.sin(angle))
            )
            # Inner points
            angle += math.pi/5
            points.append((
                center_x + size/2 * math.cos(angle),
                center_y + size/2 * math.sin(angle))
            ))
        pygame.draw.polygon(screen, HIGHLIGHT_COLOR, points)
        
    elif self.reward_type == "flower":
        # Draw a flower
        petal_colors = [(255, 150, 180), (255, 200, 150), (180, 220, 150)]
        for i in range(8):
            angle = i * math.pi/4
            px = center_x + math.cos(angle) * size
            py = center_y + math.sin(angle) * size
            pygame.draw.circle(screen, petal_colors[i % 3], (px, py), size/3)
        pygame.draw.circle(screen, HIGHLIGHT_COLOR, (center_x, center_y), size/4)
        
    elif self.reward_type == "clap":
        # Draw clapping hands
        hand_size = size/2
        # Left hand
        pygame.draw.circle(screen, (255, 220, 200), 
                          (center_x - size//2, center_y), hand_size)
        # Right hand
        pygame.draw.circle(screen, (255, 220, 200), 
                          (center_x + size//2, center_y), hand_size)
        # Clap lines
        for i in range(5):
            y_pos = center_y - hand_size//2 + i*hand_size//4
            pygame.draw.line(screen, ACCENT_COLOR, 
                           (center_x - hand_size//2, y_pos),
                           (center_x + hand_size//2, y_pos), 3)
    
    elif self.reward_type == "incorrect":
        # Draw X mark
        pygame.draw.line(screen, (255, 100, 100), 
                       (center_x - size, center_y - size),
                       (center_x + size, center_y + size), 15)
        pygame.draw.line(screen, (255, 100, 100), 
                       (center_x + size, center_y - size),
                       (center_x - size, center_y + size), 15)
    
    # Decrease timer
    self.reward_timer -= 1
    if self.reward_timer <= 0:
        self.reward_active = False

# Add the main game loop method
def run(self):
    """Main game loop"""
    running = True
    while running:
        running = self.handle_events()
        
        # Update
        if self.reward_active:
            self.draw_reward()
        
        # Draw everything
        screen.fill(BACKGROUND)
        
        # Draw UI panels
        pygame.draw.rect(screen, PANEL_BG, (0, 0, SCREEN_WIDTH, 100))
        pygame.draw.rect(screen, PANEL_BG, (0, SCREEN_HEIGHT - 100, SCREEN_WIDTH, 100))
        
        # Draw book title
        book_text = self.title_font.render(self.books[self.current_book_index], True, TEXT_COLOR)
        screen.blit(book_text, (SCREEN_WIDTH//2 - book_text.get_width()//2, 30))
        
        # Draw page info
        page_text = self.font.render(f"Page {self.current_page}/{self.total_pages} - Section {self.section}", True, TEXT_COLOR)
        screen.blit(page_text, (SCREEN_WIDTH//2 - page_text.get_width()//2, SCREEN_HEIGHT - 130))
        
        # Draw reference and drawing surfaces
        screen.blit(self.reference_surface, (10, 150))
        pygame.draw.rect(screen, TEXT_COLOR, (8, 148, SCREEN_WIDTH//2 - 16, SCREEN_HEIGHT - 246), 2)
        
        screen.blit(self.drawing_surface, (SCREEN_WIDTH//2 + 12, 150))
        pygame.draw.rect(screen, TEXT_COLOR, (SCREEN_WIDTH//2 + 10, 148, SCREEN_WIDTH//2 - 20, SCREEN_HEIGHT - 246), 2)
        
        # Draw UI labels
        ref_label = self.small_font.render("Reference", True, TEXT_COLOR)
        draw_label = self.small_font.render("Your Drawing", True, TEXT_COLOR)
        screen.blit(ref_label, (SCREEN_WIDTH//4 - ref_label.get_width()//2, 110))
        screen.blit(draw_label, (3*SCREEN_WIDTH//4 - draw_label.get_width()//2, 110))
        
        # Draw buttons
        for button in self.buttons:
            # Skip hidden buttons
            if "visible" in button and not button["visible"]:
                continue
                
            color = BUTTON_HOVER if button["rect"].collidepoint(pygame.mouse.get_pos()) else BUTTON_BG
            pygame.draw.rect(screen, color, button["rect"], border_radius=10)
            pygame.draw.rect(screen, TEXT_COLOR, button["rect"], 2, border_radius=10)
            text = self.small_font.render(button["text"], True, TEXT_COLOR)
            screen.blit(text, (button["rect"].centerx - text.get_width()//2, 
                             button["rect"].centery - text.get_height()//2))
        
        # Draw download buttons
        for button in self.download_buttons:
            color = BUTTON_HOVER if button["rect"].collidepoint(pygame.mouse.get_pos()) else button["color"]
            pygame.draw.rect(screen, color, button["rect"], border_radius=10)
            pygame.draw.rect(screen, TEXT_COLOR, button["rect"], 2, border_radius=10)
            text = self.small_font.render(button["text"], True, button["text_color"])
            screen.blit(text, (button["rect"].centerx - text.get_width()//2, 
                             button["rect"].centery - text.get_height()//2))
        
        # Draw color palette
        for button in self.color_buttons:
            pygame.draw.rect(screen, button["color"], button["rect"])
            pygame.draw.rect(screen, TEXT_COLOR, button["rect"], 2)
            # Highlight selected color
            if button["color"] == self.current_color:
                pygame.draw.rect(screen, HIGHLIGHT_COLOR, 
                               (button["rect"].x-5, button["rect"].y-5, 
                                button["rect"].width+10, button["rect"].height+10), 3)
        
        # Draw brush sizes
        for button in self.brush_buttons:
            pygame.draw.rect(screen, BUTTON_BG, button["rect"], border_radius=5)
            pygame.draw.rect(screen, TEXT_COLOR, button["rect"], 2, border_radius=5)
            # Draw brush preview
            center_x = button["rect"].centerx
            center_y = button["rect"].centery
            if self.brush_shape == "round":
                pygame.draw.circle(screen, self.current_color, (center_x, center_y), button["size"]//2)
            elif self.brush_shape == "square":
                pygame.draw.rect(screen, self.current_color, 
                               (center_x - button["size"]//2, center_y - button["size"]//2,
                                button["size"], button["size"]))
            else:  # marker
                pygame.draw.circle(screen, self.current_color, (center_x, center_y), button["size"]//2)
                # Add marker tip
                pygame.draw.rect(screen, (200, 200, 200), 
                               (center_x - button["size"]//4, center_y + button["size"]//2,
                                button["size"]//2, button["size"]))
            # Highlight selected size
            if button["size"] == self.brush_size:
                pygame.draw.rect(screen, HIGHLIGHT_COLOR, 
                               (button["rect"].x-3, button["rect"].y-3,
                                button["rect"].width+6, button["rect"].height+6), 3)
        
        # Draw brush shape selector
        shape_text = self.small_font.render("Brush Shape:", True, TEXT_COLOR)
        screen.blit(shape_text, (350, 100))
        for i, shape in enumerate(BRUSH_SHAPES):
            rect = pygame.Rect(500 + i*120, 100, 100, 40)
            color = BUTTON_HOVER if rect.collidepoint(pygame.mouse.get_pos()) else BUTTON_BG
            if shape == self.brush_shape:
                color = HIGHLIGHT_COLOR
            pygame.draw.rect(screen, color, rect, border_radius=5)
            pygame.draw.rect(screen, TEXT_COLOR, rect, 2, border_radius=5)
            text = self.small_font.render(shape.capitalize(), True, TEXT_COLOR)
            screen.blit(text, (rect.centerx - text.get_width()//2, rect.centery - text.get_height()//2))
        
        # Draw license info
        license_text = self.small_font.render(self.license_info["license"], True, TEXT_COLOR)
        screen.blit(license_text, (SCREEN_WIDTH//2 - license_text.get_width()//2, SCREEN_HEIGHT - 40))
        
        pygame.display.flip()
        self.clock.tick(60)

# Add this at the end to start the application
if __name__ == "__main__":
    app = DrawingApp()
    app.update_page()  # Initialize the first page
    app.run()