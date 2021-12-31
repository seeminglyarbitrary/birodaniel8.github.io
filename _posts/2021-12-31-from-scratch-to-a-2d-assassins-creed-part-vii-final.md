---
layout: post
title: "From scratch to a 2D Assassin's Creed game: Part VII (Final)"
author: "Daniel Biro"
categories: pygame
tags: [python, pygame, games, assassins creed]
image: 2021-12-31-header-pygame-vii.png
date: 2021-12-31
toc: true
---

2021 was a very interesting year for me for many different reasons and one of them was the start of this "blog".
The goal was to try out something new and to document some of the projects I am working on in my free time and so far I am quite enjoying it.
So let's finish this year with the final post on the Assassin's Creed Pygame project. 

### Pygame Creed 2D

As mentioned in the last post, now we have everything we need and the final objective is only to make the appearance look like a normal game.
This first involves finding a proper name for the game. I don't have any intention to make money with this so I hope there are no copyright issues, 
thus I gave it a name: Pygame Creed 2D.
```python
pygame.display.set_caption("Pygame Creed 2D")  # app title
```

### Home screen

We also need some welcoming screen with some instructions:
```python
STARTSCREEN = pygame.image.load(os.path.join("Assets", "start_screen.png"))
...
def main():
    ...
    WIN.blit(STARTSCREEN, (0, 0))  # start screen
    pygame.display.update()
    game_started = False
```

This all looks good and we initiated a `game_started` variable as false here, but the game still starts immediately so there is some extra logic needed within the main `while` loop:

```python
def main():
    while run:
        ...
        for event in pygame.event.get():
            ...
            # start the game with pressing Enter (return):
            if not game_started and event.type == pygame.KEYDOWN:
                if event.key == pygame.K_RETURN:
                    game_started = True

            if game_started:
                # moving with keys
                # guard alert
                # king found
        if game_started:
            # move player
            # move guards
            # is the player hiding
            # guard alerts
            # is the king found
            # draw window
```

Simply all the events and function calls are separated whether the game has started or not (it can be started by pressing enter).
This way the start screen doesn't get overwritten immediately and it gives a nice flow to the game.

![startpage](/assets/img/2021-12-31-start-page.png "StarPage")

Finally, I have added a `draw_text` function that renders some input text to the center of the screen with the given font (comicsans) and size (100).
This function is called both cases (guard alert and king found) when the game is over. And after a couple of seconds, the program breaks the `while` loop but the `main` function is called again, therefore we are redirected to the home screen to start a new game.

```python
def draw_text(text):
    """Draw a text to the center of the screen"""
    text_to_draw = pygame.font.SysFont("comicsans", 100).render(text, 1, WHITE)
    WIN.blit(text_to_draw, 
             (WIDTH // 2 - text_to_draw.get_width() // 2, 
              HEIGHT // 2 - text_to_draw.get_height() // 2))
    pygame.display.update()

def main():
    while run:
        ...
            # break the game if the player or a dead body was been detected:
            if event.type == GUARD_ALERT:
                draw_text("YOU HAVE BEEN DETECTED!")
                pygame.time.delay(3000)  # wait 3 sec
                run = False

            # break the game (winning!) if the king has been found:
            if event.type == KING_FOUND:
                draw_text("YOU HAVE FOUND THE KING!")
                pygame.time.delay(6000)  # wait 6 sec
                run = False
        ...
    main()

```

### Final settings

These additional lines conclude all the features I wanted to include in the game and I hope if you are reading this you also found it interesting and helpful. 
I really enjoyed building up this game and I have learned a lot about game logic. As usual, the complete code is available at my git repo here: [main_part_vi.py](https://github.com/birodaniel8/assassins_creed_2d_game/blob/main/to_blog/main_part_vi.py){:target="_blank"}, but the main folder of the repo contains the code in a more structured format. 

First, I have created a separate file to contain the game configuration (`game_config.py`) such as the base settings, input images, location of the obstacles bridges, etc. I have put the different character classes to a separate file (`character.py`) and some of the used functions (`utils.py`) as well. Finally, the `main.py` file wraps together the game (you should run this to start playing).

### Executable version

All this is fun, but I still want to be able to share this game with a friend who doesn't have programming experience. Thus, I have looked into the way to create a Windows executable version and as it turns out, it is quite easy to do by following these steps (the zip file also included in the repo):
- pip install `pyisntaller`
- run `pyisntaller main.py --onefile --noconsole` in your terminal in the folder that contains the `main.py` file
- copy the `Assets` folder to the new `dist` folder (the exe file there can start the game)
- zip the file and the assets folder

Thank you for reading through this series of posts and I wish you a Happy New Year!

![newyear](/assets/img/2021-12-31-new-year.png "NewYear")