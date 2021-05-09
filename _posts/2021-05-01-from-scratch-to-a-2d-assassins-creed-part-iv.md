---
layout: post
title: "From scratch to a 2D Assassin's Creed game: Part IV"
author: "Daniel Biro"
categories: pygame
tags: [python, pygame, games, assassins creed]
image: 2021-05-01-header-pygame-iv.png
date: 2021-05-01
toc: true
---

In this post, I am going to add the guards to the game and also getting super geeky on the applicability of some high-school math to videogames. As I mentioned before, the purpose of this game at the end will be to find a selected player (the king) without getting noticed by the guards. This means we need to add some guards to the map and give them some ability to notice our player. I will also add some bushes to the map to provide some hiding place where the player can move without being noticed.

### Bushes as hiding places

First, let's just add the bushes and the hiding functionality as it is quite similar to the behavior of the walls from the last post with some minor changes. 

After loading the bush image, we create a new Bush class and a Rect object corresponding to the bush instance. Let's also place 10 bushes on the map at a random position in the given area:
{% highlight python %}
BUSH = pygame.image.load(os.path.join("Assets", "bush.png"))
...
class Bush:
    def __init__(self, picture, x, y, size):
        self.picture, self.x, self.y, self.size = \
            picture, x, y, size
        # the bush rectangle is a third of the bush size to make the hiding more realistic:
        self.rect = pygame.Rect(self.x+self.size/3, self.y+self.size/3, self.size/3, self.size/3)
...
def main():
    bushes = [Bush(BUSH, 
                   np.random.randint(375, 400), 
                   np.random.randint(200, 450), 50) for i in range(10)]
{% endhighlight %}

Obviously, these won't appear right away because we have to draw them to the screen within our `draw_window` function, but make sure you add this segment after the drawing of the player. This way the player will hide under these bush objects:

{% highlight python %}
def draw_window(player, ... bushes):
    ...
    for bush in bushes:
        WIN.blit(reshape_and_rotate(bush.picture, bush.size, 0), (bush.x, bush.y))
{% endhighlight %}

The purpose of the bushes is to let the player hide, so I create a `hiding` attribute to the Player class and whenever the player collides with any of the bushes' rectangle object, we set this variable to `true` (similar behavior to colliding with the walls):

{% highlight python %}
player.hiding = True if np.sum([player.rect.colliderect(bush.rect) for bush in bushes]) > 0 else False
{% endhighlight %}

By testing this functionality, I have noticed that is it hard to keep up with the player's direction and movement if it moves under the bushes, therefore I have added a little arrow when the player is hiding and the arrow's rotation coincides with the player's direction (-90 degree rotation is needed because the base images have different rotation):

{% highlight python %}
if player.hiding:
        WIN.blit(reshape_and_rotate(ARROW, 35, player.rotation-90), (player.x, player.y))
{% endhighlight %}

### Guards

![guards](/assets/img/2021-05-01-guards.png "Guards")

Adding the guards to the game holds more interesting features to discuss. The first step is to load the images similar to the player's movement (but different color):
{% highlight python %}
GUARD_RIGHT = pygame.image.load(os.path.join("Assets", "guard_right.png"))
GUARD_LEFT = pygame.image.load(os.path.join("Assets", "guard_left.png"))
GUARD_STAND = pygame.image.load(os.path.join("Assets", "guard_stand.png"))
VIEW_RANGE = pygame.image.load(os.path.join("Assets", "view_range.png"))
{% endhighlight %}

The last item is a "view range", which is just a colored slightly transparent circle sector (covering 160 degrees). This component will show much the guard can see. In the final game, I will have 2 types of guards: standing (no movement) and walking. But in this post, I will only focus on the standing guard to add the base functionalities, because the walking of the guard is again not a trivial task. For this I created a general `Guard` class and one child class for the standing guard:

{% highlight python %}
class Guard(Character):
    """Guard class"""
    def __init__(self, pic, name, x, y, rotation, size):
        ...
        self.view_range_scale = 8
        self.view_range_x = self.x + self.size / 2 - self.size * self.view_range_scale / 2
        self.view_range_y = self.y + self.size / 2 - self.size * self.view_range_scale / 2

class GuardStanding(Guard):
    """Standing guard class"""
{% endhighlight %}

Note that I have already attached the position of the view range to the general guard object in a way that the center of the view range is the same as the center of the guard's rectangle object. Now I just have to add a guard to the screen the usual way:

{% highlight python %}
def main():
    ...
    guards = [GuardStanding(GUARD_STAND, name="Guard_1", x=1075, y=570, rotation=90, size=35)]
...
def draw_window(player, guards, bushes):
    ...
    for guard in guards:
        WIN.blit(reshape_and_rotate(guard.pic, guard.size, guard.rotation), (guard.x, guard.y))
        WIN.blit(reshape_and_rotate(VIEW_RANGE, guard.size * guard.view_range_scale, guard.rotation-180),
                (guard.view_range_x, guard.view_range_y))
{% endhighlight %}

Similar to the player's hiding arrow, the rotation of the view range is attached to the rotation of the guard.

![view-range](/assets/img/2021-05-01-view-range.png "View Range")


### Calculating the distance and angle between the player and a guard
All this is working just fine so far, but the player can walk into the guard's view range and nothing happens. For this, we need to check whether the player is in the view range and if so, we have to trigger some event. There are two things we need to know for this: the distance and angle between the player and the guard. If the distance is less than the radius of the view range and the angle is in that 160-degree range, the player is detected. 

Calculating the distance is straightforward, I have just used a simple coordinate system Pythagorean theorem between the center coordinates of the player and guard object's rectangle. However, the angle was not that easy and I have spent quite a lot of time figuring  it out and I had to go back to use my high-school math knowledge but now I am quite pleased by my final solution (although I am pretty sure there are other more elegant solutions out there). Let's see how it works:

![cosine-law](/assets/img/2021-05-01-cosine-law.png "Cosine Law")

Let's denote the player's location `A` and the guard's location `C`. I have created an auxiliary point `B`, 100 units from the guard, determined by its rotation. The angle I am interested in is $\gamma$. My solution is that after calculating all the side lengths of the triangle  determined by `ABC`, one can use the Cosine Law to calculate the angle:

$$ c^2 = a^2 + b^2 - 2ab cos \gamma $$

where $c = \vec{AB}$, $a = \vec{BC}$ and $b = \vec{AC}$. The following function does exactly this on determining the distance and angle between two objects and given the properties of NumPy's `arccos` function, the angle starts at 0 and goes to 180 both clockwise and anti-clockwise:

{% highlight python %}
def get_distance_and_angle(obj1, obj2):
    """Get the distance and the angle between two objects with a rectangle in their attributes using cosine law"""
    # applying cosine law:
    a_x, a_y = obj1.rect.centerx, obj1.rect.centery  # A: obj1 location
    c_x, c_y = obj2.rect.centerx, obj2.rect.centery  # C: obj2 location
    distance_ac = np.sqrt((a_x - c_x)**2 + (a_y - c_y)**2)  # distance between obj1 and obj2
    # generate an auxiliary point B at the direction of obj2 rotation with distance 100 from C:
    x_delta = -np.sin(obj2.rotation * np.pi / 180) * 100
    y_delta = -np.cos(obj2.rotation * np.pi / 180) * 100
    b_x, b_y = c_x + x_delta, c_y + y_delta  # B
    distance_bc = 100
    distance_ab = np.sqrt((a_x - b_x)**2 + (a_y - b_y)**2)
    # cosine law:
    angle = np.arccos((distance_bc**2 + distance_ac**2 - distance_ab**2) /
                      (2 * distance_bc * distance_ac)) * 180 / np.pi
    return distance_ac, angle
{% endhighlight %}

### Guard alerts as user-defined events
Now that we know how to check whether the player runs into the view range, we need some event to be triggered when this happens. Pygame has many built-in events and event listeners but the users can also define their own events. These can be particularly useful when we are in a given function or method, but we want to change a property/variable, not in scope (for example at the tutorial video I watched, when a bullet hits a spaceship, the player's remaining lives must be decreased, but that variable is not passed to the function that checks whether the spaceship was hit).

Adding a user-defined event is very easy, we just have to create a variable to use a given identifier to this user-defined event. In my case this event corresponds to the guard's alerts when the player runs into the view range:

{% highlight python %}
GUARD_ALERT = pygame.USEREVENT + 1  # these numbers are just identifiers
{% endhighlight %}

Once that is created, we can add functionality when this event happens (wait 3 sec and close the game):

{% highlight python %}
def main():
    while run:
        for event in pygame.event.get():
            ...
            if event.type == GUARD_ALERT:
                pygame.time.delay(3000)  # wait 3 sec
                run = False
{% endhighlight %}

### Triggering the alert event
There is nothing more left to do but to actually check whether the player has run into the view range and if so, trigger the above-created user event. For this, I have placed a function inside the while loop which checks this for all guards in the map and triggers the event if the player is not hiding. For this we have to `post` an `Event` with the specified user-defined identifier:

{% highlight python %}
def guard_alerts(player, guards):
    for guard in guards:
        distance, angle = get_distance_and_angle(player, guard)
        if not player.hiding and distance < 140 and angle < 80:
            pygame.event.post(pygame.event.Event(GUARD_ALERT))
...
def main():
    while run:
        ...
        guard_alerts(player, guards)
{% endhighlight %}

Now if the player runs into the highlighted view range, the game stops and closes after 3 seconds.

This post became quite long but a lot of very important functionality has been added and we are getting very close to have all major steps ready for the final properly playable version. In the next post, I will show how to make the player able to kill the guards with a little fight cloud animation.

The complete code is available at my git repo here: [main_part_iv.py](https://github.com/birodaniel8/assassins_creed_2d_game/blob/main/to_blog/main_part_iv.py){:target="_blank"}