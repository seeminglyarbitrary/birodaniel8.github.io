---
layout: post
title: "From scratch to a 2D Assassin's Creed game: Part V"
author: "Daniel Biro"
categories: pygame
tags: [python, pygame, games, assassins creed]
image: 2021-06-06-header-pygame-v.png
date: 2021-06-06
toc: true
---

Currently, in the Assassin's Creed like Pygame replica I have at this stage, the feature to have guards on the map is added and they can discover our player, however, in the real game the player can eliminate the guards too. Today, I will add the capability to kill the guards and also a tiny animation of the fight between the player and the guard to be killed.

### Two additional guard features

First, I experimented with adding a single `alive` property to the guard class, however, it became problematic when I tried to add some animation and also to change the picture of the guard to a dead man after the animation. I finally decided to add an auxiliary variable labeling the time when the guard got killed and when the animation happened I set the guard to be dead and change its appearance (it seems strange that these two doesn't mean the same, but here I separated the start of the killing process and being dead just for game logic convenience)

{% highlight python %}
class Guard(Character):
    ...
    self.alive = True
    self.killed_at = None
{% endhighlight %}

Additionally, I only show the view range if the guard has not yet been killed:

{% highlight python %}
def draw_window(player, guards, bushes):
    ...
    for guard in guards:
        ...
        # if not killed or being killed, add the view range:
        if guard.killed_at is None:
            WIN.blit(reshape_and_rotate(VIEW_RANGE, guard.size * guard.view_range_scale, guard.rotation-180),
                     (guard.view_range_x, guard.view_range_y))
{% endhighlight %}

And, only a living and not yet killed guard can alert:

{% highlight python %}
def guard_alerts(player, guards):
    for guard in guards:
        distance, angle = get_distance_and_angle(player, guard)
        if guard.alive and guard.killed_at is None:
            if not player.hiding and distance < 140 and angle < 80:
                pygame.event.post(pygame.event.Event(GUARD_ALERT))
{% endhighlight %}

### Animate the kill

The following snippet includes all the code needed to add the kill functionality to the game:

{% highlight python %}
def draw_window(player, guards, bushes):
    ...
    for guard in guards:
        if player.rect.collidepoint(guard.rect.center) > 0 and guard.alive:
            WIN.blit(reshape_and_rotate(FIGHT_CLOUD, 70, np.random.randint(180)),
                     (guard.x-guard.size/2, guard.y-guard.size/2))
            now_time = pygame.time.get_ticks()
            if guard.killed_at is None:
                guard.killed_at = now_time
            elif now_time - guard.killed_at > 200:
                guard.alive = False
                guard.pic = GUARD_DEAD
{% endhighlight %}

Let me explain. First, I check whether the player's rectangle object overlaps with the center of the guard's rectangle object using the `collidepoint` method. I used this to make the process more realistic and not to trigger the effect before the player gets close enough to the guard. If it overlaps, the guard eliminating process starts, and a `FIGHT_CLOUD` image is added at a random rotation which makes it like an animation from old fairy tales using this cloud effect when two characters fight. 

Then if the guard has not yet been killed, I am assigning the current time to the `killed_at` property and if a small amount of time passes, the guard is set to be dead and its appearance is updated to the `GUARD_DEAD` image. Notice that the animation stops if the guard is dead or if the player moves away.

This is all for today's post, but we are very close to the final version. In the next post, I will create a guard with a walking feature to make the game more exciting and in the final one I add a start screen as well and wrap up this series.

As usual, the complete code is available at my git repo here: [main_part_v.py](https://github.com/birodaniel8/assassins_creed_2d_game/blob/main/to_blog/main_part_v.py){:target="_blank"}