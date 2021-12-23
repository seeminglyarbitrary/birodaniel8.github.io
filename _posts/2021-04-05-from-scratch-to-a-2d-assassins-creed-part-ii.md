---
layout: post
title: "From scratch to a 2D Assassin's Creed game: Part II"
author: "Daniel Biro"
categories: pygame
tags: [python, pygame, games, assassins creed]
image: 2021-04-05-header-pygame-ii.png
date: 2021-04-05
toc: true
---

In this second part of creating a fully functioning AC-type game, I will describe how to make our player moving around with the `w`-`a`-`d` keys.
In the previous post, the playable character has been created and we have changed its position by pressing key `w`. Now we want to allow it to change direction and at the end, we will add some animations to the movement.
As a first step I have created a `Character` class as later on I will have additional non-playable characters like guards and I have also added the `move` method which will handle the movement of the player.

```python
class Character:
    def __init__(self, ...some_args_here...):
        ...

    def move(self, keys_pressed):
        ...
```

### Rotation
Changing the rotation parameter of the character is quite simple and very similar to the forward movement. When the key `a` or `d` is pressed, we change the rotation attribute of the character by a certain amount (`rotation_speed` controls how much the rotation is changed). One thing to bear in mind is that the rotation goes anti-clockwise and the 0 degree is at 12 o'clock:
```python
def move(self, keys_pressed):
    if keys_pressed[pygame.K_a]:
        self.rotation += self.rotation_speed
    if keys_pressed[pygame.K_d]:
        self.rotation -= self.rotation_speed
    ...
```
where `key_pressed` is the returned value from `pygame.key.get_pressed()`. Now we can just rotate the image with the `pygame.transform.rotate` function, however this function doesn't rotate the [image around its center](https://stackoverflow.com/questions/4183208/how-do-i-rotate-an-image-around-its-center-using-pygame){:target="_blank"}, hence makes the rotation quite strange. But I have managed to find a nice and handy function which rotates the images around their center and I have also created a function which wraps the reshaping and rotation steps:

```python
def rotate_around_center(image, angle):
    rotation_rect = image.get_rect().copy()
    rotation_image = pygame.transform.rotate(image, angle)
    rotation_rect.center = rotation_image.get_rect().center
    rotation_image = rotation_image.subsurface(rotation_rect).copy()
    return rotation_image

def reshape_and_rotate(img, size, rotation):
    return rotate_around_center(
        pygame.transform.scale(img, (size, size)), rotation)
```

### Moving forward in different angles

The forward movement of the player is quite straightforward if the angle can be divided by 90 as we just add or substitute to the x or y coordinates. However, when the angle cannot be divided by 90, the movement is quite tricky as we must add some to both the x and y coordinates. I also want that the distance the player moves is the same for each direction. So this was the step where I had to refresh my high-school knowledge on the unit circle and trigonometric functions. Moving the same distance to each direction means that from the current position I will move to the corresponding point of the unit circle and the change in the x and y coordinates can be calculated by sine and cosine functions with some additional transformations given by Pygame:

![sin_cos](/assets/img/2021-04-05-cos-sin.png "Sine-Cosine movement")

First, we have to rotate the coordinate system by 90 degrees anti-clockwise as the 0 degrees in pygame is at 12 o'clock. Second, we need to look at the right-bottom corner as the increments of x and y correspond to these directions in pygame. Finally, we have to convert the angle to radians ($rad = deg \times \pi / 180$) and multiply the coordinate changes by the speed of the movement:
```python
def move(self, keys_pressed):
    ...
    self.x -= np.sin(self.rotation * np.pi / 180) * self.speed  
    self.y -= np.cos(self.rotation * np.pi / 180) * self.speed
    ...
```

### Animation

To make the movement of the walking player complete I have also added some animation. As for now, the player is actually moving but we only see a top-view standing person picture. I have downloaded two additional steps of the walking process with right or left leg forward steps. 

![animation](/assets/img/2021-04-05-animation.png "Animation")

However, we cannot just switch between these images in each iteration because then the game would update these images 60 times in a second (FPS). And I also wanted to switch back to the standing image when the movement is over. This latter is quite easy I only had to add a new event to the event loop in main():

```python
while run:
    ...
    for event in pygame.event.get():
        ...
        if event.type == pygame.KEYUP:
            if event.key == pygame.K_w:
                character_0.pic = WALKING_STAND
        ...
```

These few lines will change the `pic` attribute of the `character_0` object (player object) to the standing top-view image if the key `w` has been released (keyup event).

The more interesting part is the actual animation during the walk: the idea is to start measuring the time from the start of the movement (keydown event on `w`) and update the images at some given frequency. This frequency can be tied to the character's speed to make the animation more realistic. I also want the animation and the appearance of the images to be periodic, therefore I applied the modulo(4) operator on the passed time to get a 4 element cycle (right leg forward - stand - left leg forward - stand):

```python
while run:
    ...
    for event in pygame.event.get():
        ...
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_w:
                character_0.walk_start_time = pygame.time.get_ticks()
        ...
```

```python
def move(self):
    ...
    if keys_pressed[pygame.K_w]:
        time_now = pygame.time.get_ticks()
        time_diff = (time_now - self.walk_start_time) / (500 / self.speed)
        time_diff_mod = np.mod(int(time_diff), 4)
        if time_diff_mod == 0:
            self.pic = WALKING_RIGHT
        elif (time_diff_mod == 1) or (time_diff_mod == 3):
            self.pic = WALKING_STAND
        else:
            self.pic = WALKING_LEFT
    ...
```

Although I only had 3 different images to animate the movement, this is now working just fine. But one last thing I have added to my code is to make the walking a bit less smooth. With the current implementation, it sometimes looks as if the player is sliding on the screen. To make the location change a bit less smooth I only update the location of the player if a certain time has passed:

```python
def move(self):
    ...
    if (pygame.time.get_ticks() - self.walk_last_time > (FPS * 0.6)):
        self.x -= np.sin(self.rotation * np.pi / 180) * self.speed  
        self.y -= np.cos(self.rotation * np.pi / 180) * self.speed
        self.walk_last_time = pygame.time.get_ticks()
    ...
```

This small adjustment makes very little difference but one can adjust the smoothness changing the 0.6 parameter to something else.

That's it for this post. Oh, and I added a cool GTA like background (`WIN.blit(BACKGROUND, (0, 0))`) to the game. Next time I will add some obstacles to the map to make sure the player cannot just walk anywhere on the screen.

The complete code is available at my git repo here: [main_part_ii.py](https://github.com/birodaniel8/assassins_creed_2d_game/blob/main/to_blog/main_part_ii.py){:target="_blank"}

### Useful resources:
- [How do I rotate an image around its center using Pygame?](https://stackoverflow.com/questions/4183208/how-do-i-rotate-an-image-around-its-center-using-pygame){:target="_blank"}
- [Walking movement viewed from above](https://www.drawinghowtodraw.com/stepbystepdrawinglessons/wp-content/uploads/2016/05/howtodraw-figure-people-running-08.jpg){:target="_blank"}