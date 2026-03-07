+++
title = "Level Up Your C: Why Raylib is the Best Way for Beginners to Learn C"
date = "2026-01-21T22:30:00+05:30"
author = "Kanthi"
authorTwitter = "kanthi"
cover = "/images/raylib/raylib_animation.png"
tags = ["c", "raylib", "gamedev", "programming", "learning", "graphics"]
keywords = ["raylib", "c programming", "game development for beginners", "graphics programming", "learn c", "c tutorial"]
description = "Stop staring at the terminal. Learn C by building games and visual applications with raylib—a simple, powerful, and fun library perfect for beginners."
showFullContent = false
readingTime = true
hideComments = false
Toc = true
draft = true
+++

Most people start learning C by writing programs that live entirely in the terminal. While `printf("Hello, World!\n");` is a rite of passage, it doesn't take long before the black-and-white text becomes a bit... boring.

If you've ever felt like C is too "low-level" or "difficult" to do anything fun with, I have a secret for you: **raylib**.

---

## What is Raylib?

[raylib](https://www.raylib.com/) is a simple and easy-to-use library to enjoy videogames programming. It's written in pure C and designed specifically for learners, prototyping, and tools development, but it's powerful enough for professional games and visualization.

Created by **Ramon Santamaria** (better known as [@raysan5](https://github.com/raysan5)), raylib is:

- 🎮 **Simple**: Designed with education in mind
- ⚡ **Fast**: Written in pure C with minimal dependencies
- 🌍 **Cross-platform**: Works on Windows, macOS, Linux, Raspberry Pi, and even Web (via Emscripten)
- 📚 **Well-documented**: Over 140 examples included
- 🆓 **Free and Open Source**: Licensed under zlib/libpng

> **Fun Fact**: The name "raylib" is a nod to the old **Borland BGI** (Borland Graphics Interface) and **XNA framework**. It brings that same spirit of "just make graphics work" to modern C programming.

---

## Why is Raylib Perfect for Learning C?

Here's why I think raylib is the **best tool** for beginners to learn C programming:

### 1. 🎯 Immediate Visual Feedback

Instead of tracking variables in a debugger or reading text output, you **see your code come alive**. If your logic is wrong, your character flies off into space. If your math is off, the shapes look weird. It makes debugging intuitive and even fun.

### 2. 🔧 No "Magic"

Unlike heavy engines like Unity or Unreal, raylib is just a library. You still write a standard `main()` function. You still manage your own memory. You still control the "Game Loop." It reinforces C fundamentals rather than hiding them.

### 3. 📐 Procedural Style

Raylib uses a flat, procedural API. You don't need to understand complex Object-Oriented patterns to get a circle on the screen. You just call `DrawCircle()`. This aligns perfectly with how C works.

### 4. 🚀 Zero Hassle Setup

It has almost no dependencies. Setting it up is usually a single command:

| Platform | Command |
|----------|---------|
| **macOS** | `brew install raylib` |
| **Linux (Debian/Ubuntu)** | `sudo apt install libraylib-dev` |
| **Linux (Fedora)** | `sudo dnf install raylib-devel` |
| **Windows** | Download from [raylib releases](https://github.com/raysan5/raylib/releases) |

### 5. 📖 The Game Loop = Real-World Programming

Every raylib program follows a pattern: **initialize → update → draw → cleanup**. This teaches you:

- State management
- Frame-rate control
- Input handling
- Memory management (textures, sounds, etc.)

These are universal programming concepts that transfer to any domain.

---

## Getting Started: Installation

### macOS (Homebrew)

```bash
brew install raylib
```

### Linux (Debian/Ubuntu)

```bash
sudo apt update
sudo apt install build-essential libraylib-dev
```

### Compiling Your First Program

Once installed, you can compile a raylib program like this:

```bash
# macOS
gcc main.c -o game $(pkg-config --libs --cflags raylib)

# Or with all flags explicit
gcc main.c -o game -lraylib -lm -lpthread -ldl -framework OpenGL -framework Cocoa -framework IOKit
```

```bash
# Linux
gcc main.c -o game -lraylib -lGL -lm -lpthread -ldl -lrt -lX11
```

---

## Example 1: The "Hello World" of Graphics

Let's start with the most basic raylib program—opening a window and displaying text.

```c
#include "raylib.h"

int main(void) {
    // Initialization
    InitWindow(800, 450, "raylib [core] example - basic window");
    SetTargetFPS(60);  // Set our game to run at 60 frames-per-second

    // Main game loop
    while (!WindowShouldClose()) {  // Detect window close button or ESC key
        // Update: Game logic goes here (none for now)

        // Draw
        BeginDrawing();
            ClearBackground((Color){ 26, 27, 38, 255 }); // Dark background
            DrawText("Hello, raylib!", 300, 200, 20, WHITE);
        EndDrawing();
    }

    // De-Initialization
    CloseWindow();  // Close window and OpenGL context
    return 0;
}
```

**Save as `main.c`, compile, and run!** You'll see:

![Your first raylib window](/images/raylib/01-hello-window.png)

### Breaking Down the Code

| Code | What It Does |
|------|--------------|
| `InitWindow(800, 450, "title")` | Creates a window 800×450 pixels with a title |
| `SetTargetFPS(60)` | Caps the frame rate to 60 FPS |
| `WindowShouldClose()` | Returns `true` if user clicked close or pressed ESC |
| `BeginDrawing()` / `EndDrawing()` | Starts and ends a drawing frame |
| `ClearBackground(RAYWHITE)` | Fills the screen with a color |
| `DrawText(...)` | Draws text at specified position |
| `CloseWindow()` | Cleans up and closes |

---

## Example 2: Drawing Basic Shapes

Let's draw some shapes! Raylib makes 2D graphics incredibly simple.

```c
#include "raylib.h"

int main(void) {
    InitWindow(800, 450, "raylib - Basic Shapes");
    SetTargetFPS(60);

    while (!WindowShouldClose()) {
        BeginDrawing();
            ClearBackground((Color){ 26, 27, 38, 255 });

            // Draw a filled circle
            DrawCircle(100, 100, 50, RED);

            // Draw a filled rectangle
            DrawRectangle(600, 50, 150, 100, BLUE);

            // Draw a line
            DrawLine(50, 225, 750, 225, WHITE);

            // Draw a triangle
            DrawTriangle(
                (Vector2){ 400, 400 },  // Point 1
                (Vector2){ 350, 300 },  // Point 2
                (Vector2){ 450, 300 },  // Point 3
                GOLD
            );

            // You can also draw outlines
            DrawCircleLines(400, 150, 40, DARKGRAY);
            DrawRectangleLines(250, 50, 100, 80, PURPLE);

        EndDrawing();
    }

    CloseWindow();
    return 0;
}
```

![Basic shapes in raylib](/images/raylib/02-basic-shapes.png)

### Available Shape Functions

| Function | Description |
|----------|-------------|
| `DrawCircle(x, y, radius, color)` | Filled circle |
| `DrawCircleLines(x, y, radius, color)` | Circle outline |
| `DrawRectangle(x, y, width, height, color)` | Filled rectangle |
| `DrawRectangleLines(x, y, width, height, color)` | Rectangle outline |
| `DrawLine(x1, y1, x2, y2, color)` | Straight line |
| `DrawTriangle(v1, v2, v3, color)` | Filled triangle |
| `DrawPoly(center, sides, radius, rotation, color)` | Regular polygon |

---

## Example 3: Animating a Bouncing Ball

Now let's add movement! This example shows the core game loop pattern.

```c
#include "raylib.h"

int main(void) {
    // Initialization
    const int screenWidth = 800;
    const int screenHeight = 450;

    InitWindow(screenWidth, screenHeight, "raylib - Bouncing Ball");
    SetTargetFPS(60);

    // Ball properties
    float ballX = 400.0f;
    float ballY = 225.0f;
    float ballSpeedX = 5.0f;
    float ballSpeedY = 4.0f;
    float ballRadius = 20.0f;

    while (!WindowShouldClose()) {
        // UPDATE: Move the ball
        ballX += ballSpeedX;
        ballY += ballSpeedY;

        // Bounce off walls
        if (ballX >= (screenWidth - ballRadius) || ballX <= ballRadius) {
            ballSpeedX *= -1.0f;  // Reverse X direction
        }
        if (ballY >= (screenHeight - ballRadius) || ballY <= ballRadius) {
            ballSpeedY *= -1.0f;  // Reverse Y direction
        }

        // DRAW
        BeginDrawing();
            // Dark nice background
            ClearBackground((Color){ 20, 20, 40, 255 });

            // Draw the ball
            DrawCircle((int)ballX, (int)ballY, ballRadius, RED);

            // Display position
            DrawText(TextFormat("Ball Position: X: %.0f Y: %.0f", ballX, ballY),
                     20, screenHeight - 40, 20, WHITE);
        EndDrawing();
    }

    CloseWindow();
    return 0;
}
```

![Bouncing ball animation](/images/raylib/03-bouncing-ball.png)

### Key Concepts Here

1. **Game State Variables**: `ballX`, `ballY`, `ballSpeedX`, `ballSpeedY` store the game state
2. **Update Phase**: Movement logic happens before drawing
3. **Collision Detection**: Simple bounds checking
4. **`TextFormat()`**: Like `printf()` but returns a string for drawing

---

## Example 4: Keyboard Input

Let's make something interactive! Move a square with the arrow keys or WASD.

```c
#include "raylib.h"

int main(void) {
    const int screenWidth = 800;
    const int screenHeight = 450;

    InitWindow(screenWidth, screenHeight, "raylib - Keyboard Input");
    SetTargetFPS(60);

    // Player properties
    float playerX = 400.0f;
    float playerY = 225.0f;
    const float playerSize = 50.0f;
    const float playerSpeed = 5.0f;

    while (!WindowShouldClose()) {
        // UPDATE: Handle keyboard input
        if (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D)) playerX += playerSpeed;
        if (IsKeyDown(KEY_LEFT) || IsKeyDown(KEY_A))  playerX -= playerSpeed;
        if (IsKeyDown(KEY_DOWN) || IsKeyDown(KEY_S))  playerY += playerSpeed;
        if (IsKeyDown(KEY_UP) || IsKeyDown(KEY_W))    playerY -= playerSpeed;

        // Keep player inside screen bounds
        if (playerX < 0) playerX = 0;
        if (playerX > screenWidth - playerSize) playerX = screenWidth - playerSize;
        if (playerY < 0) playerY = 0;
        if (playerY > screenHeight - playerSize) playerY = screenHeight - playerSize;

        // DRAW
        BeginDrawing();
            ClearBackground((Color){ 26, 27, 38, 255 });  // Dark gray (custom color)

            DrawText("Use WASD or Arrow Keys to Move", 220, 50, 24, WHITE);

            // Draw the player
            DrawRectangle((int)playerX, (int)playerY, (int)playerSize, (int)playerSize, RED);

            // Display position
            DrawText(TextFormat("Position: X: %.0f Y: %.0f", playerX, playerY),
                     250, screenHeight - 50, 20, WHITE);
        EndDrawing();
    }

    CloseWindow();
    return 0;
}
```

![Keyboard input example](/images/raylib/04-keyboard-input.png)

### Input Functions

| Function | Description |
|----------|-------------|
| `IsKeyDown(KEY_X)` | Returns `true` while key is held |
| `IsKeyPressed(KEY_X)` | Returns `true` only on the frame the key was pressed |
| `IsKeyReleased(KEY_X)` | Returns `true` only on the frame the key was released |
| `GetKeyPressed()` | Returns the last key pressed |

---

## Example 5: Mouse Input

Let's follow the mouse cursor and respond to clicks!

```c
#include "raylib.h"

int main(void) {
    const int screenWidth = 800;
    const int screenHeight = 450;

    InitWindow(screenWidth, screenHeight, "raylib - Mouse Input");
    SetTargetFPS(60);

    Color ballColor = BLUE;
    Vector2 screenCenter = { screenWidth / 2.0f, screenHeight / 2.0f };

    while (!WindowShouldClose()) {
        // Get mouse position
        Vector2 mousePos = GetMousePosition();

        // Change color on left click
        if (IsMouseButtonPressed(MOUSE_BUTTON_LEFT)) {
            // Cycle through colors
            if (ColorToInt(ballColor) == ColorToInt(BLUE)) ballColor = RED;
            else if (ColorToInt(ballColor) == ColorToInt(RED)) ballColor = GREEN;
            else if (ColorToInt(ballColor) == ColorToInt(GREEN)) ballColor = GOLD;
            else if (ColorToInt(ballColor) == ColorToInt(GOLD)) ballColor = PURPLE;
            else ballColor = BLUE;
        }

        // DRAW
        BeginDrawing();
            ClearBackground((Color){ 26, 27, 38, 255 });

            // Draw line from center to mouse
            DrawLineEx(screenCenter, mousePos, 2.0f, DARKGRAY);

            // Draw circle at mouse position
            DrawCircle((int)mousePos.x, (int)mousePos.y, 30, ballColor);

            // Display info
            DrawText(TextFormat("Mouse X: %.0f Y: %.0f", mousePos.x, mousePos.y),
                     20, 20, 20, DARKGRAY);
            DrawText("Click to change color", 550, 20, 20, DARKGRAY);
        EndDrawing();
    }

    CloseWindow();
    return 0;
}
```

![Mouse input example](/images/raylib/05-mouse-input.png)

### Mouse Functions

| Function | Description |
|----------|-------------|
| `GetMousePosition()` | Returns `Vector2` with current mouse position |
| `GetMouseX()` / `GetMouseY()` | Returns individual coordinates |
| `IsMouseButtonDown(button)` | Is button currently held? |
| `IsMouseButtonPressed(button)` | Was button just pressed? |
| `GetMouseWheelMove()` | Returns scroll wheel movement |

---

## Example 6: Beautiful Gradients

Raylib has built-in gradient drawing functions that make beautiful backgrounds easy.

```c
#include "raylib.h"

int main(void) {
    const int screenWidth = 800;
    const int screenHeight = 450;

    InitWindow(screenWidth, screenHeight, "raylib - Gradients");
    SetTargetFPS(60);

    int gradientType = 0;

    while (!WindowShouldClose()) {
        // Cycle through gradient types with SPACE
        if (IsKeyPressed(KEY_SPACE)) {
            gradientType = (gradientType + 1) % 4;
        }

        BeginDrawing();
            // Draw different gradient backgrounds
            switch (gradientType) {
                case 0:
                    DrawRectangleGradientV(0, 0, screenWidth, screenHeight,
                                           DARKBLUE, PURPLE);
                    DrawText("Vertical Gradient", 300, 200, 30, WHITE);
                    break;
                case 1:
                    DrawRectangleGradientH(0, 0, screenWidth, screenHeight,
                                           MAROON, ORANGE);
                    DrawText("Horizontal Gradient", 280, 200, 30, WHITE);
                    break;
                case 2:
                    DrawRectangleGradientEx(
                        (Rectangle){ 0, 0, screenWidth, screenHeight },
                        RED, BLUE, GREEN, YELLOW);
                    DrawText("Four-Corner Gradient", 270, 200, 30, WHITE);
                    break;
                case 3:
                    DrawRectangleGradientV(0, 0, screenWidth, screenHeight,
                                           (Color){ 25, 25, 112, 255 },   // Midnight Blue
                                           (Color){ 255, 105, 180, 255 }); // Hot Pink
                    DrawText("Beautiful Gradients in C", 230, 200, 30, WHITE);
                    break;
            }

            DrawText("Press SPACE to change gradient", 260, screenHeight - 40, 20, WHITE);
        EndDrawing();
    }

    CloseWindow();
    return 0;
}
```

![Beautiful gradients](/images/raylib/06-gradient.png)

---

## Example 7: Fun with Animation (Many Moving Objects)

Let's create a more dynamic scene with multiple animated objects.

```c
#include "raylib.h"
#include <stdlib.h>

#define MAX_CIRCLES 30

typedef struct {
    float x;
    float y;
    float speedX;
    float speedY;
    float radius;
    Color color;
} Circle;

int main(void) {
    const int screenWidth = 800;
    const int screenHeight = 450;

    InitWindow(screenWidth, screenHeight, "raylib - Animation Example");
    SetTargetFPS(60);

    // Initialize circles with random properties
    Circle circles[MAX_CIRCLES];
    Color colors[] = { RED, GREEN, BLUE, YELLOW, ORANGE, PURPLE, SKYBLUE, PINK, LIME, GOLD };

    for (int i = 0; i < MAX_CIRCLES; i++) {
        circles[i].x = GetRandomValue(50, screenWidth - 50);
        circles[i].y = GetRandomValue(50, screenHeight - 50);
        circles[i].speedX = GetRandomValue(-4, 4);
        circles[i].speedY = GetRandomValue(-4, 4);
        if (circles[i].speedX == 0) circles[i].speedX = 2;
        if (circles[i].speedY == 0) circles[i].speedY = 2;
        circles[i].radius = GetRandomValue(10, 30);
        circles[i].color = colors[GetRandomValue(0, 9)];
    }

    while (!WindowShouldClose()) {
        // UPDATE: Move all circles
        for (int i = 0; i < MAX_CIRCLES; i++) {
            circles[i].x += circles[i].speedX;
            circles[i].y += circles[i].speedY;

            // Bounce off walls
            if (circles[i].x <= circles[i].radius ||
                circles[i].x >= screenWidth - circles[i].radius) {
                circles[i].speedX *= -1;
            }
            if (circles[i].y <= circles[i].radius ||
                circles[i].y >= screenHeight - circles[i].radius) {
                circles[i].speedY *= -1;
            }
        }

        // DRAW
        BeginDrawing();
            ClearBackground((Color){ 26, 27, 38, 255 });  // Tokyo Night background

            // Draw all circles
            for (int i = 0; i < MAX_CIRCLES; i++) {
                DrawCircle((int)circles[i].x, (int)circles[i].y,
                          circles[i].radius, circles[i].color);
            }

            DrawText(TextFormat("FPS: %d", GetFPS()), 10, 10, 20, WHITE);
            DrawText("Bouncing Circles Animation", 260, screenHeight - 30, 20, LIGHTGRAY);
        EndDrawing();
    }

    CloseWindow();
    return 0;
}
```

![Raylib Animation Example](/images/raylib/07-animation.png)

### Key Concepts Here

- **Structs**: Organizing related data together (position, speed, color)
- **Arrays**: Managing multiple game objects
- **Random Values**: `GetRandomValue(min, max)` for procedural generation
- **`GetFPS()`**: Displays current frame rate

---

## Example 8: Simple UI Buttons

Let's create interactive buttons—useful for menus and tools.

```c
#include "raylib.h"

// Button structure
typedef struct {
    Rectangle bounds;
    const char *text;
    Color normalColor;
    Color hoverColor;
    Color pressedColor;
    bool isHovered;
    bool isPressed;
} Button;

// Check button state
void UpdateButton(Button *btn) {
    Vector2 mouse = GetMousePosition();
    btn->isHovered = CheckCollisionPointRec(mouse, btn->bounds);
    btn->isPressed = btn->isHovered && IsMouseButtonPressed(MOUSE_BUTTON_LEFT);
}

// Draw button
void DrawButton(Button btn) {
    Color color = btn.normalColor;
    if (btn.isHovered) color = btn.hoverColor;
    if (IsMouseButtonDown(MOUSE_BUTTON_LEFT) && btn.isHovered) color = btn.pressedColor;

    DrawRectangleRounded(btn.bounds, 0.3f, 10, color);

    // Center text in button
    int textWidth = MeasureText(btn.text, 24);
    DrawText(btn.text,
             btn.bounds.x + (btn.bounds.width - textWidth) / 2,
             btn.bounds.y + (btn.bounds.height - 24) / 2,
             24, WHITE);
}

int main(void) {
    const int screenWidth = 800;
    const int screenHeight = 450;

    InitWindow(screenWidth, screenHeight, "raylib - UI Buttons");
    SetTargetFPS(60);

    // Create buttons
    Button startBtn = {
        .bounds = { 300, 180, 200, 50 },
        .text = "START GAME",
        .normalColor = (Color){ 76, 175, 80, 255 },   // Green
        .hoverColor = (Color){ 102, 187, 106, 255 },  // Light green
        .pressedColor = (Color){ 56, 142, 60, 255 },  // Dark green
    };

    Button exitBtn = {
        .bounds = { 300, 250, 200, 50 },
        .text = "EXIT",
        .normalColor = (Color){ 244, 67, 54, 255 },   // Red
        .hoverColor = (Color){ 239, 83, 80, 255 },    // Light red
        .pressedColor = (Color){ 198, 40, 40, 255 },  // Dark red
    };

    const char *message = "";

    while (!WindowShouldClose()) {
        // UPDATE
        UpdateButton(&startBtn);
        UpdateButton(&exitBtn);

        if (startBtn.isPressed) message = "Game Starting!";
        if (exitBtn.isPressed) message = "Goodbye!";

        // DRAW
        BeginDrawing();
            ClearBackground((Color){ 26, 27, 38, 255 });  // Dark background

            DrawText("Simple Menu", 310, 100, 40, WHITE);

            DrawButton(startBtn);
            DrawButton(exitBtn);

            // Show message
            if (message[0] != '\0') {
                DrawText(message, 330, 330, 24, DARKGRAY);
            }
        EndDrawing();

        // Exit if exit button pressed
        if (exitBtn.isPressed) break;
    }

    CloseWindow();
    return 0;
}
```

![Raylib GUI Buttons Example](/images/raylib/08-buttons.png)

### Concepts Covered

- **Custom structs** for UI components
- **Collision detection**: `CheckCollisionPointRec()` - is mouse over rectangle?
- **Visual feedback**: Different colors for hover/press states
- **Button rounded corners**: `DrawRectangleRounded()`
- **Text centering**: Using `MeasureText()` to calculate width

---

## Example 9: Building Pong (A Complete Mini-Game!)

Let's put it all together and build the classic Pong game!

```c
#include "raylib.h"

int main(void) {
    // Screen dimensions
    const int screenWidth = 800;
    const int screenHeight = 450;

    InitWindow(screenWidth, screenHeight, "raylib - Pong Game");
    SetTargetFPS(60);

    // Paddle properties
    const float paddleWidth = 10.0f;
    const float paddleHeight = 80.0f;
    const float paddleSpeed = 6.0f;

    // Player 1 (left)
    float player1Y = (screenHeight - paddleHeight) / 2;

    // Player 2 (right - AI controlled)
    float player2Y = (screenHeight - paddleHeight) / 2;

    // Ball properties
    float ballX = screenWidth / 2.0f;
    float ballY = screenHeight / 2.0f;
    float ballSpeedX = 5.0f;
    float ballSpeedY = 4.0f;
    const float ballSize = 10.0f;

    // Scores
    int score1 = 0;
    int score2 = 0;

    while (!WindowShouldClose()) {
        // UPDATE
        // Player 1 controls (W/S keys)
        if (IsKeyDown(KEY_W) && player1Y > 0) player1Y -= paddleSpeed;
        if (IsKeyDown(KEY_S) && player1Y < screenHeight - paddleHeight) player1Y += paddleSpeed;

        // Player 2 controls (Arrow keys) - or simple AI
        if (IsKeyDown(KEY_UP) && player2Y > 0) player2Y -= paddleSpeed;
        if (IsKeyDown(KEY_DOWN) && player2Y < screenHeight - paddleHeight) player2Y += paddleSpeed;

        // Simple AI for single player (follows ball)
        // Uncomment for AI opponent:
        // float targetY = ballY - paddleHeight / 2;
        // if (player2Y < targetY) player2Y += paddleSpeed * 0.8f;
        // if (player2Y > targetY) player2Y -= paddleSpeed * 0.8f;

        // Move ball
        ballX += ballSpeedX;
        ballY += ballSpeedY;

        // Ball collision with top/bottom
        if (ballY <= 0 || ballY >= screenHeight - ballSize) {
            ballSpeedY *= -1;
        }

        // Ball collision with paddles
        Rectangle leftPaddle = { 20, player1Y, paddleWidth, paddleHeight };
        Rectangle rightPaddle = { screenWidth - 30, player2Y, paddleWidth, paddleHeight };
        Rectangle ballRect = { ballX, ballY, ballSize, ballSize };

        if (CheckCollisionRecs(ballRect, leftPaddle) && ballSpeedX < 0) {
            ballSpeedX *= -1.1f;  // Slight speed increase
        }
        if (CheckCollisionRecs(ballRect, rightPaddle) && ballSpeedX > 0) {
            ballSpeedX *= -1.1f;
        }

        // Scoring
        if (ballX < 0) {
            score2++;
            ballX = screenWidth / 2;
            ballY = screenHeight / 2;
            ballSpeedX = 5.0f;
        }
        if (ballX > screenWidth) {
            score1++;
            ballX = screenWidth / 2;
            ballY = screenHeight / 2;
            ballSpeedX = -5.0f;
        }

        // DRAW
        BeginDrawing();
            ClearBackground((Color){ 26, 27, 38, 255 });

            // Draw center line
            for (int i = 0; i < screenHeight; i += 20) {
                DrawRectangle(screenWidth / 2 - 2, i, 4, 10, WHITE);
            }

            // Draw paddles
            DrawRectangle(20, (int)player1Y, (int)paddleWidth, (int)paddleHeight, WHITE);
            DrawRectangle(screenWidth - 30, (int)player2Y, (int)paddleWidth, (int)paddleHeight, WHITE);

            // Draw ball
            DrawRectangle((int)ballX, (int)ballY, (int)ballSize, (int)ballSize, WHITE);

            // Draw scores
            DrawText(TextFormat("%d", score1), screenWidth / 4, 30, 60, WHITE);
            DrawText(TextFormat("%d", score2), 3 * screenWidth / 4 - 30, 30, 60, WHITE);

            // Instructions
            DrawText("P1: W/S    P2: UP/DOWN", 290, screenHeight - 30, 16, GRAY);
        EndDrawing();
    }

    CloseWindow();
    return 0;
}
```

![Raylib Pong Game Example](/images/raylib/09-pong.png)

### What You've Learned Here

- **Rectangle collision**: `CheckCollisionRecs()` for paddle-ball collision
- **Game state management**: Tracking positions, speeds, and scores
- **Scoring and reset**: Resetting game state when scoring
- **Two-player input**: Handling multiple input schemes

---

## Example 10: Loading and Drawing Textures (Images)

Real games use images. Here's how to load and draw them:

```c
#include "raylib.h"

int main(void) {
    const int screenWidth = 800;
    const int screenHeight = 450;

    InitWindow(screenWidth, screenHeight, "raylib - Loading Textures");
    SetTargetFPS(60);

    // Load texture from file
    // Create a simple image file or use any PNG/JPG
    Texture2D texture = LoadTexture("sprite.png");

    float rotation = 0.0f;
    float scale = 1.0f;

    while (!WindowShouldClose()) {
        // Animate
        rotation += 1.0f;
        scale = 1.0f + 0.2f * sinf(rotation * DEG2RAD);

        BeginDrawing();
            ClearBackground((Color){ 26, 27, 38, 255 });

            if (texture.id > 0) {
                // Draw texture with rotation and scaling
                DrawTextureEx(texture,
                    (Vector2){ screenWidth/2 - texture.width*scale/2,
                               screenHeight/2 - texture.height*scale/2 },
                    rotation, scale, WHITE);
            } else {
                DrawText("Could not load sprite.png", 260, 200, 20, RED);
                DrawText("Place a PNG file called 'sprite.png' in the same folder", 180, 230, 16, GRAY);
            }

            DrawText(TextFormat("Loaded: sprite.png (%dx%d)", texture.width, texture.height),
                     250, screenHeight - 40, 20, WHITE);
        EndDrawing();
    }

    // IMPORTANT: Unload texture when done
    UnloadTexture(texture);

    CloseWindow();
    return 0;
}
```

![Raylib Texture Loading Example](/images/raylib/10-texture.png)

### Texture Functions

| Function | Description |
|----------|-------------|
| `LoadTexture("file.png")` | Load image file as texture |
| `UnloadTexture(texture)` | Free texture memory (important!) |
| `DrawTexture(tex, x, y, tint)` | Draw texture at position |
| `DrawTextureEx(tex, pos, rotation, scale, tint)` | Draw with transformations |
| `DrawTexturePro(...)` | Advanced drawing with source/destination rectangles |

---

## Example 11: 3D Graphics in C!

Yes, raylib can do 3D too! Here's a simple 3D cube:

```c
#include "raylib.h"

int main(void) {
    const int screenWidth = 800;
    const int screenHeight = 450;

    InitWindow(screenWidth, screenHeight, "raylib - 3D Cube");
    SetTargetFPS(60);

    // Define camera
    Camera3D camera = { 0 };
    camera.position = (Vector3){ 5.0f, 5.0f, 5.0f };    // Camera position
    camera.target = (Vector3){ 0.0f, 0.0f, 0.0f };      // Looking at
    camera.up = (Vector3){ 0.0f, 1.0f, 0.0f };          // Up vector
    camera.fovy = 45.0f;                                 // Field of view
    camera.projection = CAMERA_PERSPECTIVE;

    float rotation = 0.0f;

    while (!WindowShouldClose()) {
        // Rotate camera around the cube
        rotation += 0.5f;
        camera.position.x = sinf(rotation * DEG2RAD) * 6.0f;
        camera.position.z = cosf(rotation * DEG2RAD) * 6.0f;

        BeginDrawing();
            // Dark background
            ClearBackground((Color){ 26, 27, 38, 255 });

            BeginMode3D(camera);
                // Draw a colored cube
                DrawCube((Vector3){ 0.0f, 0.0f, 0.0f }, 2.0f, 2.0f, 2.0f, RED);
                DrawCubeWires((Vector3){ 0.0f, 0.0f, 0.0f }, 2.0f, 2.0f, 2.0f, GREEN);

                // Draw grid on the floor
                DrawGrid(10, 1.0f);
            EndMode3D();

            DrawText("3D in C is Easy!", 30, 40, 30, WHITE);
            DrawText("Camera rotating around the cube", 250, screenHeight - 30, 20, LIGHTGRAY);
        EndDrawing();
    }

    CloseWindow();
    return 0;
}
```

![Raylib 3D Cube Example](/images/raylib/11-3d-cube.png)

### 3D Basics

| Concept | Description |
|---------|-------------|
| `Camera3D` | Defines viewpoint (position, target, up vector) |
| `BeginMode3D()` / `EndMode3D()` | Switch to 3D rendering mode |
| `DrawCube()` / `DrawCubeWires()` | Draw solid/wireframe cube |
| `DrawSphere()` / `DrawCylinder()` | Other 3D primitives |
| `DrawGrid()` | Helpful reference grid |

---

## Compilation Cheat Sheet

### macOS (with Homebrew)

```bash
gcc -o game main.c $(pkg-config --libs --cflags raylib)
./game
```

### Linux

```bash
gcc -o game main.c -lraylib -lGL -lm -lpthread -ldl -lrt -lX11
./game
```

### Using Make (Recommended)

Create a `Makefile`:

```makefile
CC = gcc
CFLAGS = -Wall -Wextra
LDFLAGS = $(shell pkg-config --libs --cflags raylib)

game: main.c
	$(CC) $(CFLAGS) -o $@ $< $(LDFLAGS)

clean:
	rm -f game

.PHONY: clean
```

Then just run:

```bash
make
./game
```

---

## Where to Go Next?

### Official Resources

- 📚 **[raylib Cheatsheet](https://www.raylib.com/cheatsheet/cheatsheet.html)** - All functions at a glance
- 📖 **[raylib Examples](https://www.raylib.com/examples.html)** - 140+ examples with source code
- 📘 **[raylib Wiki](https://github.com/raysan5/raylib/wiki)** - Guides and tutorials
- 💬 **[raylib Discord](https://discord.gg/raylib)** - Active community

### Project Ideas for Learning

| Difficulty | Project |
|------------|---------|
| ⭐ Beginner | Bouncing DVD logo, clicking game |
| ⭐⭐ Easy | Snake, Breakout, Flappy Bird clone |
| ⭐⭐⭐ Medium | Platformer, Space Invaders, Asteroids |
| ⭐⭐⭐⭐ Hard | Roguelike, puzzle game, 3D maze |

---

## Conclusion

If you're learning C and want to make it **fun**, raylib is the answer. It gives you:

✅ **Visual feedback** instead of boring text
✅ **Real-world patterns** like game loops and state management
✅ **Motivation** to keep coding because you're making cool stuff
✅ **Foundation** that transfers to any graphics/game programming

Stop staring at `printf` outputs. **Draw something. Move something. Build something.**

Happy coding! 🎮

---

> **About raylib**: raylib is developed by Ramon Santamaria ([@raysan5](https://github.com/raysan5)) and a wonderful community of contributors. It's completely free and open source under the zlib/libpng license.
