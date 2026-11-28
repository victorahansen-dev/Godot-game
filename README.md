# Notes - what I have learnt so far and my game progression

## 1. Player
```GdScript
extends CharacterBody2D

#variables/constants
const SPEED = 300.0

var last_direction: Vector2 = Vector2.RIGHT
var is_attacking: bool = false
var hitbox_offset: Vector2
var strength: int = 20


@onready var animated_sprite_2d: AnimatedSprite2D = $AnimatedSprite2D
@onready var swing_sword: AudioStreamPlayer2D = $SwingSword
@onready var hitbox: Area2D = $Hitbox



func _ready() -> void: # function is only used once when the node is created
	
	#initialise hitbox offset
	hitbox_offset = hitbox.position

func _physics_process(_delta: float) -> void:
	# disables hitbox until an attack is triggered
	hitbox.monitoring = false
	
	if Input.is_action_just_pressed("attack") and not is_attacking:
		attack()
		
	# skip movement if attacking
	if is_attacking:
		velocity = Vector2.ZERO
		return
	
	process_movement() # in the physics_process we call for the process_movement function
	process_animation() # and we pass that into the process_animation function
	# that function checks the speed of the player and determines if they're either walking or idle
	# based off either one of those, the play_animation function will run either "walk" or "idle" and then calls the appropriate animation
	move_and_slide()

# ------------------------
# Movement & animation
# ------------------------
func process_movement() -> void:
	# Get the input direction and handle the movement/deceleration.
	var direction := Input.get_vector("left", "right", "up", "down") # once that is done, we get the direction the player is moving in
	
	if direction != Vector2.ZERO: # if direction is not zero (the player is moving)
		velocity = direction * SPEED # we will move the player around
		last_direction = direction  # and we will store the last direction which will be set as the current direction
		update_hitbox_offset()
	else: # if the direction is zero (the player is not moving)
		velocity = Vector2.ZERO
	
	
	
func process_animation() -> void:
	if is_attacking:
		return
	if velocity != Vector2.ZERO: # if player is not stationary - moving in any direction at all
		play_animation("walk", last_direction) # play the walk animations
	else:
		play_animation("idle", last_direction)

# void is used for when a function does not return anything
func play_animation(prefix: String, dir: Vector2) -> void: # prefix runs whatever is passed in, either that be idle or walk
	if dir.x != 0:
		animated_sprite_2d.flip_h = dir.x < 0
		animated_sprite_2d.play(prefix + "_right")
	elif dir.y < 0:
		animated_sprite_2d.play(prefix + "_up")
	elif dir.y > 0:
		animated_sprite_2d.play(prefix + "_down")


# ------------------------
# Attacking
# ------------------------

func attack() -> void: # attack function
	is_attacking = true # attacking is true when this function is running
	hitbox.monitoring = true # turns monitoring for collision on
	swing_sword.play() # plays the sword swing audio
	play_animation("attack", last_direction) # plays the sword swing animation, also uses last_direction to choose the appropriate animation frame

func _on_animated_sprite_2d_animation_finished() -> void:
	if is_attacking:
		is_attacking = false

# ------------------------
# Hitbox
# ------------------------

func update_hitbox_offset() -> void:
	var x := hitbox_offset.x
	var y := hitbox_offset.y
	
	match last_direction: # match gives you a more compact version of if and elif statements while performing the same action
		Vector2.LEFT:
			hitbox.position = Vector2(-x, y)
		Vector2.RIGHT:
			hitbox.position = Vector2(x, y)
		Vector2.UP:
			hitbox.position = Vector2(y, -x)
		Vector2.DOWN:
			hitbox.position = Vector2(-y, x)


func _on_hitbox_body_entered(body: Node2D) -> void: # the reason we use body for hitbox_entered is because slime is a CharacterBody2D node, simply put it is a body
	if is_attacking and body.name.begins_with("Slime"): # prints the name of all bodies that begin with "Slime", alongside printing "Hit".
		body.take_damage(strength, position)
```
---

**Here:**

This script controls a 2D player character: movement, directional animations, sword attacks, sound effects, and positioning the attack hitbox.
Variables and node references

- SPEED sets the player’s movement speed.

- last_direction remembers the most recent movement direction. It’s used to choose which way the player faces when idle or attacking.

- is_attacking tracks whether an attack animation is in progress.

- hitbox_offset stores the hitbox’s starting position, so the script can reposition it around the player.

- strength is the damage value passed to an enemy when the attack connects.

- The @onready variables reference nodes in the player scene: the animated sprite, swing sound, and hitbox.

## Setup

_ready() runs when the player enters the scene. It saves the hitbox’s initial position in hitbox_offset.
Movement and animation

_physics_process() runs on physics frames. It checks for an attack input, stops normal movement while attacking, otherwise processes movement and animation, then calls move_and_slide() to move the character.

process_movement() reads the directional inputs with Input.get_vector(). When there’s input, it sets the player’s velocity and remembers the direction. With no input, it sets velocity to zero.

process_animation() chooses a walk animation when the player is moving and an idle animation otherwise. Both use last_direction, so the player keeps facing the last direction they moved.

play_animation() chooses an animation based on the direction:

Horizontal directions use the right-facing animation, with flip_h mirroring it for left.
Vertical directions use the up or down animation.

## Attacking

attack() marks the player as attacking, enables the hitbox, plays the swing sound, and starts an attack animation in the last movement direction.

When the sprite’s animation finishes, _on_animated_sprite_2d_animation_finished() clears is_attacking, allowing movement and another attack.
Hitbox and damage

update_hitbox_offset() moves the hitbox based on the player’s facing direction, using the saved starting offset as a reference.

When a body enters the hitbox, _on_hitbox_body_entered() checks that an attack is active and that the body’s name starts with "Slime". It then calls take_damage() on that body, passing the player’s strength and position.
Things to check

Hitbox timing: _physics_process() sets hitbox.monitoring = false at the start of every physics frame. attack() turns it on, but the next physics frame turns it off again. That may leave the hitbox active for only a brief moment. If you want it active for the whole attack, enable it when the attack starts and disable it when the attack animation finishes—or use a dedicated hit frame or animation event.

Hitbox positioning: update_hitbox_offset() is called when the player moves, but not when they start an attack. If the player attacks while standing still after turning another way, make sure the hitbox is already positioned correctly. You could update it when the facing direction changes or at the start of attack().

Diagonal directions: Input.get_vector() can return diagonal directions. match last_direction only handles exact Vector2.LEFT, RIGHT, UP, and DOWN values, so a diagonal may not match any case. If you want four-direction movement and attacks, convert the input to a cardinal direction before storing it.

Signal connections and enemy method: The sprite’s animation_finished signal needs to be connected to _on_animated_sprite_2d_animation_finished(). Any body identified as a slime also needs a take_damage(strength, position) method.

Directional hitbox offsets: The four Vector2 calculations rotate or flip the original offset around the player. Check that the initial offset and the hitbox shape’s dimensions make the attack area land where you expect in each direction.


"@onready var animated_sprite_2d: AnimatedSprite2D = $AnimatedSprite2D" is a variable created which gives access to the sprite animations

in physics_process we call for the process_movement and process_animation function

process_movement checks the speed of the player and determines if they're either walking or idle

based off either one of those, the play_animation function will run either "walk" or "idle" and then calls the appropriate animation

---

# Extra notes

## Collision configuration
In the Godot engine, layers can be used to tell where something is in the physics layers, sort of as a categorization method.

While mask defines specifically which body/object can collide with each other, this being determined by the number each node is placed inside of.

An example of this; if our player is on layer 2, but mask one. It will collide with everything else that also is in mask one.

Additionally, in this game specifically where we use a sword to attack, the mask of the hitbox itself must match the layer of the enemy it is attacking.

This way, when the sword is swung, it will be checking for collision with anything in layer 2 - because the sword hitbox is in mask 2.

Side note: this assumes that your node has a CollisionShape, which is what allows for collision inside Godot in the first place.

## Functions
"void" is used when a function is not returning anything


## Differentiating between functions

In GDScript, a leading underscore is a convention, not an access restriction: it often marks a function as internal or not intended to be called from outside the script. 

It also appears on engine callbacks such as _ready() and _process(), which Godot calls automatically at specific points.

Simply put, underscores can be used as a marker/indicator on custom functions (primarily to signify for them to only be used inside the one specific script), they are also used for engine callbacks built into GDScript.

Thus, no underscore generally signifies that the function be used over multiple scripts.

While this isn't necessary, it's useful if you work with others on the same script and/or like to have better flow over your code.
