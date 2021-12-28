---
layout: post
title: "From scratch to a 2D Assassin's Creed game: Part VI"
author: "Daniel Biro"
categories: pygame
tags: [python, pygame, games, assassins creed]
image: 2021-11-30-header-pygame-vi.png
date: 2021-11-30
toc: true
---

It has been a couple of months since I posted anything here due to being busy with some changes but now I would like to finish up the series on my little Python game.
In this post, I will add guards who can move around the map to detect our player and also the king that is the target character. Let's start with the latter one.

### The king

The role of the king is very simple in the game, it just stands at a single point until our hero finds him, and then the mission is accomplished.
```python
KING = pygame.image.load(os.path.join("Assets", "king.png"))
...
def draw_window(player, guards, bushes):
    ...
    WIN.blit(reshape_and_rotate(KING, 40, 270), (50, 275))
```

The mechanism to find the king and trigger an event is very similar to the way the guard alert has been implemented.
An extra user-defined event is created and we call a function in each iteration of the running game to check whether the player is close to the king. 
If so, the user-defined event is triggered and the game is over:

```python
KING_FOUND = pygame.USEREVENT + 2 # these numbers are just identifiers
...
def king_found(player):
    """King is found event"""
    if player.rect.colliderect(pygame.Rect(40, 270, 60, 40)):
        pygame.event.post(pygame.event.Event(KING_FOUND))
...

def mean():
    while run:
        for event in pygame.event.get():
            ...
            # break the game (winning!) if the king has been found:
            if event.type == KING_FOUND:
                pygame.time.delay(6000)  # wait 3 sec
                run = False
        ...
        # is the king found:
        king_found(player)
```

### Moving guards

In the previous posts, I have created a general `Guard` class with a `GuardStanding` child to add guards that are standing but not moving on a map.
Killing these guards is quite easy now, the player just has to be close to them from behind. To make the game more exciting, I have decided to add some guards who are walking on the map. 

They are walking in a straight line back and forth from their starting position to their target position either vertically or horizontally.
These restrictions are made to make the implementation simple but still to have some dynamic elements in the game. 
In the implementation we have to differentiate these cases and set extra variables for the moving mechanism when the guard instances are created:
```python
class GuardWalking(Guard):
    """Walking guard class"""

    def __init__(self, pic, name, x, y, rotation, size, speed, moving_direction, target):
        super().__init__(pic, name, x, y, rotation=rotation, size=size)
        # the walking guard will walk between (x1, y1) and (x2, y2) with some speed
        # the walking is implemented in a way that it can only move to horizontal or vertical direction
        if moving_direction == "horizontal":
            assert np.abs(target - x) > 100, "The distance between the end points must be larger than 100"
            self.x_1, self.y_1, self.x_2, self.y_2 = x, y, target, y
            self.target_rotation = 90 if self.x_1 > self.x_2 else 270
        elif moving_direction == "vertical":
            assert np.abs(target - y) > 100, "The distance between the end points must be larger than 100"
            self.x_1, self.y_1, self.x_2, self.y_2 = x, y, x, target
            self.target_rotation = 0 if self.y_1 > self.y_2 else 180
        else:
            print("Not a valid walking direction")
        self.speed = speed

        # these are auxiliary variables for the guard movement:
        self.at_targets = True
        self.at_start = True  # if we are just at the start we don't want the target_rotation switched
```

If the moving direction is `horizontal`, the `target` parameter sets the `x` coordinate of the target, while the `vertical` case sets the `y` coordinate.
The `target_rotation` describes the direction from the starting point to the target based on these two points in space.
The guard movement also needs a speed and I set this speed to be the same as the player's default speed. 
Thus the moving guard can be killed by running towards it from behind.
```python
GuardWalking(GUARD_STAND, name="Guard_3", x=1040, y=250, rotation=0,
             size=35, speed=3, moving_direction="vertical", target=400)
```

### Guard walking

The `move` method is a little bit messy and seems complicated (and I am probably not using always the most elegant solutions) but let me try to make some sense of it:
```python
class GuardWalking(Guard):
    def move(self):
        """Movement of the walking guard"""
        if self.killed_at is None:
            # rotate the guard if it is not at the target_rotation:
            if np.mod(self.rotation, 360) != self.target_rotation:
                self.rotation += self.speed
```

First, we wrap everything in an if condition because the moving is relevant only if the guard is not being killed. 
Then if the rotation of the guard is not aligned with the target rotation I make some adjustments to the rotation until it turns to the right angle.

```python
            else:
                if self.walk_start_time is None:
                    self.walk_start_time = pygame.time.get_ticks()  # moving
                # animation:
                time_now = pygame.time.get_ticks()
                time_diff = (time_now - self.walk_start_time) / (500 / self.speed)
                time_diff_mod = np.mod(int(time_diff), 4)  # 3 possible states rotate in a 4 element cycle
                if time_diff_mod == 0:
                    self.pic = GUARD_RIGHT
                elif (time_diff_mod == 1) or (time_diff_mod == 3):
                    self.pic = GUARD_STAND
                else:
                    self.pic = GUARD_LEFT
                if (pygame.time.get_ticks() - self.walk_last_time > (FPS * 0.6)):
                    # move if the guard is in the proper direction:
                    if self.target_rotation == 270:
                        self.x += self.speed
                    elif self.target_rotation == 90:
                        self.x -= self.speed
                    elif self.target_rotation == 180:
                        self.y += self.speed
                    else:
                        self.y -= self.speed
                    self.rect.x = self.x
                    self.rect.y = self.y
                    # move the view range too:
                    self.view_range_x = self.x + self.size / 2 - self.size * self.view_range_scale / 2
                    self.view_range_y = self.y + self.size / 2 - self.size * self.view_range_scale / 2
                    self.walk_last_time = pygame.time.get_ticks()
```

If the right angle is set we start moving. The first section is the mechanism to handle the animation of the guard with the same 3 state process as the one that has been implemented for the player's movement ([here](https://github.com/birodaniel8/assassins_creed_2d_game/blob/main/to_blog/main_part_ii.py)). While the second section covers the movement of the coordinates themselves based on the moving direction. After these familiar solutions let's turn to some walking guard-specific lines.

Currently we can turn the guard to the right angle and start walking towards its target but there is nothing stopping it from just walking off the screen.
We need something that tells the guard that it arrived to the end point, now let's turn back:
```python
            if self.at_start == False and self.at_targets == False and \
                    (self.rect.collidepoint(self.x_1, self.y_1) or self.rect.collidepoint(self.x_2, self.y_2)):
                self.target_rotation = np.mod(self.target_rotation + 180, 360)
                self.at_targets = True
                self.walk_start_time = None
                self.pic = GUARD_STAND
```

For this, we need some extra variables to check that the guard is not at the start point or the target already (that it is walking towards these points).
And then if it hits one of these points we make it turn 180 degrees by increasing the target rotation by 180 and settting the `at_targets` variable to true (therefore it only changes the target rotation once). The `walk_start_time` is also set to None for restarting the animation when the guard is ready to move again. Finally, we need some additional lines to detect if the guard has left the endpoints:
```python
            if (np.sqrt((self.x - self.x_1)**2 + (self.y - self.y_1)**2) > 50) and \
                    (np.sqrt((self.x - self.x_2)**2 + (self.y - self.y_2)**2) > 50):
                self.at_targets = False
                self.at_start = False
```

Each time the distance from these points is measured and if it is larger than 50 we can say we have left these points (this also means that the endpoints must be at least 101 distance away from each other). This completes the `move` method of the GuardWalking class.

### Guards finding dead guards

One last element to make the game a bit more exciting I have also implemented another logic that enables the guards to detect already eliminated guards. Thus our hero must perform its kills in those locations where the already active guards don't notice or else the game is over. I have added this functionality to the `guard_alerts` functions where I am also iterating through all the guard pairs and checking if a guard (who is still alive) can see a killed guard within its view range. If so, a guard alert event is posted just as if the guard had noticed the player. The solution is clearly not scalable as the number of guards is getting bigger but having just a few guards, it is good enough.
```python
def guard_alerts(player, guards):
    for guard in guards:
        if guard.alive and guard.killed_at is None:
            ...
            for other_guard in guards:
                if other_guard != guard:
                    distance, angle = get_distance_and_angle(other_guard, guard)
                    # if the other guard is dead and within view range:
                    if other_guard.killed_at is not None and distance < 140 and angle < 80:
                        pygame.event.post(pygame.event.Event(GUARD_ALERT))
```

At this point, we have all the necessary building blocks for the game and now it is just a question of design where to put each character on the map. In the next (and also final) post I am going to tidy up the game a little and also make a Windows executable version of it.

The complete code is available at my git repo here: [main_part_vi.py](https://github.com/birodaniel8/assassins_creed_2d_game/blob/main/to_blog/main_part_vi.py){:target="_blank"}