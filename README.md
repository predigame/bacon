Making Bacon
============

Here's a simple maze game that demonstrates some cool things we can do with actors. Create a new file named `bacon.py` and start with our standard first three lines of code:

```python
WIDTH = 30
HEIGHT = 18
TITLE = 'Making Bacon'
```
Like before, these lines doesn't do much just yet. Here's a line to add a simple **background** (*remember backgrounds are retrieved from the backgrounds/ directory*)

```python
# use a grass background
BACKGROUND = 'grass'
```
Next we'll define a constant for the number of piggies to create. This line of code will not have any impact just yet.

```python
# how many piggies to create
PIGGIES = 10
```
Now let's create a maze with stone images. Remember that we must have a `stone` in image in our images directory.

```python
maze(callback=partial(image, 'stone'))
```
**CHECKPOINT** - try saving and running your code at this point. You should see something like the below picture.

![alt text](http://predicate.us/predigame/images/maze_grass.png "Bacon 1")

Alternatively, it's also possible to create a maze with just black rectangles instead of stone images.

```python
maze(callback=partial(shape, RECT, BLACK))
```

If all looks good with the maze, let's add our player actor. We're going for `Soldier-2`. The actor will be assigned to move on keyboard arrows and every keyboard movement will call the `evaluate` callback. This is code that we can use to "evaluate" each move to make sure the player can't walk through walls.

```python
# a callback that keeps the player from running
# into walls.
def evaluate(action, sprite, pos):
    obj = at(pos)
    if obj and obj.tag == 'wall':
        return False
    else:
        return True

# create a soldier on the bottom left grid cell
player = actor('Soldier-2', (0, HEIGHT-1), tag='player', abortable=True)

# have the solider attach to the keyboard arrows
# each move is "evaluated" to make sure the player
# doesn't walk through the wall
player.keys(precondition=evaluate)

# player moves at a speed of 5 with an animation rate of 2
# which flips the sprite image every other frame
player.speed(5).rate(2).move_to((0, HEIGHT-1))
```
Let's give this a shot. Try running the code to ensure that the player can navigate the maze maze without being able to walk through walls.

The Predigame platform includes an abstraction for respecting maze walls, so defining an `evaluate` function isn't necessary. Instead you an have the line:

```
# have the solider attach to the keyboard arrows
# each move is "evaluated" to make sure the player
# doesn't walk through the wall
player.keys(precondition=player_physics)
```

If this works, try adding some piggies. Here is one case where the `PIGGIES` constant will be used.

```python
# create a piggy function
def create_piggy(num):
    for x in range(num):
        pos = rand_pos()
        piggy = actor('Piggle', pos, tag='piggy')
        piggy.move_to((pos))
        # graze is a random walk
        piggy.wander(partial(graze, piggy), time=0.75)

# create some piggies
create_piggy(PIGGIES)
```
`Piggle` is an animated actor. There is a special callback function `wander` that allows the computer to control the movement of an actor. The `wander` is called for every movement, in the case of our piggy, calls the Predigame internal `graze` callback function which is a random and undirected movement.  

Try running saving these updates and running the game. We'll see the maze, our player, and now some piggies! It's possible to speed up the piggies by adjusting the `time` attribute for the random walk.

![alt text](http://predicate.us/predigame/images/maze_actors.png "Bacon 2")

The final part of our game is shooting! We'll create a `shoot` callback that is assigned to the space bar. It looks a little like an air shot, but the piggy effects are pretty cool.

```python
# shoot a weapon
def shoot():
    player.act(SHOOT, loop=1)

    #find the next object that is facing the player
    target = player.next_object()

    # if it's a piggy and that piggy is alive
    if target and target.tag == 'piggy' and target.health > 0:
        # kill the piggy
        target.kill()
        # tally the kill
        score(1)

    # check to see if there are any piggys left
    if len(get('piggy')) == 0:
        text('Time for some BACON!! (%s secs)' % time(), color=BLACK)
        gameover()

# register space to shoot
keydown('space', shoot)

#we're keeping score
score()

# register the 'r' key for resetting the game
keydown('r', reset)
```

Notice we use the `PIGGIES` constant a second time? That's why we made it a constant. Our game needs to track all the piggies and will stop playing when `PIGGIES` piggies have been killed. *NOTE: this code will not work if more points are assigned for each dead piggy!*
