---
layout: post
title: "From scratch to a 2D Assassin's Creed game: Part III"
author: "Daniel Biro"
categories: pygame
tags: [python, pygame, games, assassins creed]
image: 2021-04-08-header-pygame-iii.png
date: 2021-04-08
toc: true
---

Today's post is going to be a quite short one but an important feature has been added to the game, namely some barriers to restricting the movement of the playable character.
First of all, I have created a proper map, a new background for the game. I have used a [map creator app](http://heroescommunity.com/viewthread.php3?TID=43775){:target="_blank"}, which is used to create maps for Heroes III (one of the favorite games of my childhood) and now the player is on an island. In the final version, the player is deployed on one part of the island and has to find a king without being noticed by the guards, similar to the classic AC games.

### Player class and the run feature
In the previous post, I have created a class `Character` and included the method which moves our player. I have realized that only the playable character can be moved by the user, so I have redefined the Character class just to have the basic properties and inherited a `Player` class that now contains the `move` method. For the inclusion of the obstacles I will need a rectangle object associated with the player object, therefore I added a `Rect` object to the Character class properties too:

{% highlight python %}
class Character:
    def __init__(self, ...some_args_here...):
        ...
        self.rect = pygame.Rect(self.x, self.y, self.size, self.size)

class Player(Character):
    def move(self, keys_pressed):
        ...
{% endhighlight %}

In the game, I also want to allow the player to move faster if the user wants that as a classic running feature. This can be achieved very easily by adding a new key up/down event (on LeftShift) which changes the player's speed attribute.

{% highlight python %}

if event.type == pygame.KEYDOWN:
    ...
    if event.key == pygame.K_LSHIFT:
        player.speed *= 2  # running

if event.type == pygame.KEYUP:
    ...
    if event.key == pygame.K_LSHIFT:
        player.speed /= 2  # stop running

{% endhighlight %}

### Obstacle objects

Now we want to restrict the possible moving territory of the player, namely I don't want the character to be able to run out of the island into the water. I have also added some walls to the map as obstacles and bridges between the two parts but obviously, I want to let the player go through the latter one. Adding the obstacles involves creating some `Rect` objects which define the restricted places and every time we move the code should check whether we want to enter these territories. If the next movement doesn't overlap with any of these rectangles then the player can move. 

Fortunately, the Pygame's [Rect](https://www.pygame.org/docs/ref/rect.html){:target="_blank"} object is prepared to solve such problems with its `colliderect` method which returns a boolean on whether the two Rect objects overlap or not. We also came prepared by creating a Rect object for the player object, therefore the change we have to make is to check within the `move` method whether the new position of the player's Rect object does collide with any of the obstacle objects. If it doesn't then make the move but if it does then just keep the player standing at the previous position:

{% highlight python %}

OBSTACLES = [
    ...
    pygame.Rect(644, 430, 49, 237),
    ...
]

# within the "move" method:
...
x_delta = -np.sin(self.rotation * np.pi / 180) * self.speed
y_delta = -np.cos(self.rotation * np.pi / 180) * self.speed

# create a temporary rectangle with the new positions and check if the move is valid:
temp_rect = pygame.Rect(self.rect.x+x_delta, self.rect.y+y_delta, self.size, self.size)
num_collides = np.sum([temp_rect.colliderect(obstacle) for obstacle in OBSTACLES])
# if valid then move, else just stand at that point:
if num_collides == 0:
    self.rect.x += x_delta
    self.rect.y += y_delta
    self.x = self.rect.x
    self.y = self.rect.y
    self.walk_last_time = pygame.time.get_ticks()
else:
    self.pic = PLAYER_STAND
{% endhighlight %}

As far as I know, these rectangle objects as their name suggest can only cover rectangles on the map. Therefore, if you have a complicated map you have to add these rectangles one by one figuring out their size and position. It can be a painful process, but along with these small but important changes we already have a quite neat "game" where we can walk and run around an island without falling into the water.

![map](/assets/img/2021-04-08-map.png "Map")

The complete code is available at my git repo here: [main_part_iii.py](https://github.com/birodaniel8/assassins_creed_2d_game/blob/main/to_blog/main_part_iii.py){:target="_blank"}

### Useful resources:
- [Pygame's Rect object documentation](https://www.pygame.org/docs/ref/rect.html){:target="_blank"}
- [Heroes of Might and Magic 3: Map Editor](http://heroescommunity.com/viewthread.php3?TID=43775){:target="_blank"}