# Multithreaded Space Invaders

A Java Swing remake of Space Invaders built around concurrency. Game logic, mode management, and audio each run on their own thread, separate from the Swing UI thread, and every shared piece of game state is accessed under synchronization.

Built by a three-person team for an Object-Oriented Programming course at Texas State University (Spring 2026), starting from instructor-provided base code.

<!-- Add a GIF of the game running here: ![Gameplay](docs/gameplay.gif) -->

## Features

- **Two game modes:** Survival (three hearts, game over when they run out) and Creative (no damage), switchable while the game is running
- **Customizable look:** preset or custom images for the invaders and the shooter
- **Bullet types:** Lazer, TNT, and Nuke, chosen from the menu bar
- **Background music:** selectable tracks that play on their own audio thread
- **Heads-up display:** remaining hearts and the current mode

## Architecture

The game runs on four threads:

| Thread | Class | Responsibility |
| --- | --- | --- |
| Swing event thread | `SpaceInvadersUI`, `ListenerActions` | Keyboard input and repainting (Swing `Timer`, every 20 ms) |
| `GameCalculator` | `service/GameCalculator` | Movement, spawning, and collision detection |
| `ModeManager-Thread` | `service/ModeManager` | Switches between Survival and Creative mode on request |
| `MusicPlayer-Thread` | `service/MusicService` | Loads and loops audio clips |

**Thread safety:** shared game state is read and written inside `synchronized (game)` blocks. Flags that cross threads, such as the running state, a pending mode toggle, and the current mode, are `volatile`. Mode changes are requested with a flag and applied by the mode thread, which keeps input handling from racing the game loop.

**Design:**
- `GameMode` is an interface implemented by `SurvivalMode` and `CreativeMode`, so a new mode can be added without changing existing code (Open/Closed Principle).
- `BulletService` swaps bullet rendering strategies (drawn shape or image) at runtime.
- Code is split by responsibility into `ui`, `actions`, `characters`, `service`, and `exception` packages (Single Responsibility Principle).

## Controls

| Key | Action |
| --- | --- |
| Left / Right arrow | Move the shooter |
| Space (hold) | Fire |

Use the menu bar to change the game mode, images, bullet type, and music.

## Running it

Requires Java 17 or later.

**IntelliJ IDEA:** open the project, mark `src` as the sources root, and run `spaceinvaders.Main`.

**Command line**, from the repository root:

```bash
javac -d out $(find src -name "*.java")
cp -R src/spaceinvaders/resources out/spaceinvaders/
java -cp out spaceinvaders.Main
```

## My contributions

- Designed the `GameMode` interface and implemented Survival and Creative modes
- Built `ModeManager` as a dedicated thread so mode toggles happen safely without racing the game loop
- Implemented hearts that are removed on each lost life, synchronized with the game state
- Added game start, game over, and graceful shutdown of the background threads
- Added the behavior when invaders reach the shooter, fixed the shooter's hitbox, and moved mode classes into the `service` package

## Team

- Ruben Luna: game modes, mode thread, hearts, game start and game over
- Samyog Shrestha: package structure, menu bar, music service, bullet types
- Joshua Chan: invader and shooter image menus, custom image handling
