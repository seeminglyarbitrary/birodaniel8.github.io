---
layout: post
title: "From scratch to a 2D Assassin's Creed game: Part I"
author: "Daniel Biro"
categories: pygame
tags: [python, pygame, games, assassins creed]
image: 2021-03-27-header-pygame.png
date: 2021-03-27
toc: true
---

I have just recently discovered pygame, an amazingly fun package in Python by watching [this](https://www.youtube.com/watch?v=jO6qQDNa2UY) quick tutorial. In my view, it is a perfect package if you just got into programming to create one of your first little projects, but it is also a lot of fun if you consider yourself a bit more experienced programmer as I do. It is an excellent choice to apply those base programming elements you learned previously to something tangible and also to learn game logic and just genuinely enjoy the game you came up with. I am personally a huge fan of the Assassin's Creed games, although I haven't played with the newer ones, I decided to create my little game with pygame which resembles the AC games but obviously much more basic. The idea is to have a guy that we can control and a target to reach and kill. Between you and your target, there should be a bunch of guards that you have to avoid. And all this in top-view and 2D.

When I am writing this post I am still far away from the final version of the game, but in this part let's just get into the basics of the package. Pygame is a framework written as a Python package to create simple event-based games. Basically, you have to create a window that will pop up once you run your main file and write functions that will handle the events such as a key being pressed.

### Creating a window for the game

Once you loaded the package, creating a window object is very simple, the only thing you need is to set the size of it:

{% highlight python %}
import pygame
WIDTH, HEIGHT = 500, 500
WIN = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Walking man")  # app title
{% endhighlight %}

If you run your script (simply `ctrl+f5` in VSCode), you will notice that a window will pop up and disappear immediately and terminates the run. This is because although the window was created, nothing keeps it up and running. The most common solution is to have a `while` loop running until you exit the game. This loop will be active throughout the whole game and checks the events happening:

{% highlight python %}
def main():
    run = True
    while run:
        # this is where the events are handled

if __name__ == '__main__':
    main() # call main() when the script has been run
{% endhighlight %}

This `while` loop is now running as fast as it can on your computer which is not necessary so let's set the FPS of the game with pygame's Clock object:
{% highlight python %}
def main():
    clock = pygame.time.Clock()
    run = True
    while run:
        clock.tick(FPS)
{% endhighlight %}

Inside the loop, we can handle the pygame events. With some simple lines, you can get all the events and loop through them one by one. A good idea is to add a feature that quits our game when the player clicks on the exit button:
{% highlight python %}
import sys
for event in pygame.event.get():
    # quit event:
    if event.type == pygame.QUIT:
        pygame.quit()
        sys.exit()
{% endhighlight %}

The exit function from the sys package is also necessary to close the application and not let any other section to run because it would cause an error when you close the game.

### Images and rectangles

In pygame, you can easily load and display pictures by loading them and then calling the window object's `blit` method (usually the additional media files are stored in a subfolder). I am going to use a top-view image of a man standing as this will be used in the game later on. Pygame has built-in functions to also rescale and rotate your pictures:
{% highlight python %}
import os
# load an image from the Assets folder and rescale it to 50x50 pixels and rotate it 180 degrees
WALKING_STAND = pygame.transform.rotate(
    pygame.transform.scale(
        pygame.image.load(os.path.join("Assets",
         "walking_stand.png")), (50, 50)), 180)
{% endhighlight %}

Now we just have to call the window object's `blit` method and specify the location to display this object, however, pygame has some additional useful objects such as rectangles (Rect). These objects are useful as you can define interactions between them, therefore the location of the images to display is often tied to these kinds of objects. An additional important thing to know is the coordinate system used in pygame as it is quite different from the one you might used to. In pygame, the (0, 0) coordinate is the top-left corner of the window and the `x` coordinate increases to the right and the `y` coordinate increases to the bottom: 

{:.center}
![coord](/assets/img/2021-03-27-coordinate-system.png "Coordinate System")


Armored with this knowledge, displaying the images tied to a rectange is very straightforward. But to make the images actually appear we also have to update the display:
{% highlight python %}
def main():
    ...
    # rectangle with x, y, width and height parameters:
    character_rect = pygame.Rect(100, 100, 50, 50)
    # the location can be defined by the rectangle's coordinates:
    WIN.blit(WALKING_STAND, (character_rect.x, character_rect.y))
    pygame.display.update()
    ...
{% endhighlight %}

### Moving the character's image

Now we can add another event type to move our character in the game. We have to call for all the pressed keys and check whether the key we want to use is pressed (represented in a `K_` format). In this case, we only want to increase the y coordinate value of the rectangle (to move down) when the `w` key is pressed and re-draw our image:

{% highlight python %}
for event in pygame.event.get():
    ...
    # move the image:
    if pygame.key.get_pressed()[pygame.K_w]:
        character_rect.y += 1
        WIN.blit(WALKING_STAND,
                 (character_rect.x, character_rect.y))
        pygame.display.update()
    ...
{% endhighlight %}

If you run your python file now and try to move your character by pressing w, you will see something like this:

![moving-an-image](/assets/img/2021-03-27-move-image.png "Moving an image")

This is because pygame works in a way that it draws these items onto the already existing ones and once you start moving you basically draw the image again and again with slight differences in position. This is of course not ideal, but it can be easily fixed by filling the background every time you move the image and the background will hide the previously drawn images:

{% highlight python %}
for event in pygame.event.get():
    ...
    WIN.fill((255, 255, 255))  # color is given by RGB
    WIN.blit(WALKING_STAND, (character_rect.x, character_rect.y))
    pygame.display.update()
    ...
{% endhighlight %}

These are the basic ideas of pygame that one needs to have to create a more sophisticated game later on. The complete code is available at my git repo here: [main_part_i.py](https://github.com/birodaniel8/assassins_creed_2d_game/blob/main/to_blog/main_part_i.py)

### Useful resources:
- [Pygame in 90 Minutes - For Beginners](https://www.youtube.com/watch?v=jO6qQDNa2UY) 