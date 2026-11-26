# Notes - what I have learnt so far and my game progression

## 1. Animations & Movement
```GdScript
extends CharacterBody2D

const SPEED = 300.0

@onready var animated_sprite_2d: AnimatedSprite2D = $AnimatedSprite2D

var last_direction: Vector2 = Vector2.RIGHT

func _physics_process(_delta: float) -> void:
	
	process_movement() 
	process_animation() 
	move_and_slide()

# ------------------------
# Movement & animation
# ------------------------
func process_movement() -> void:
	# Get the input direction and handle the movement/deceleration.
	var direction := Input.get_vector("left", "right", "up", "down")
	
	if direction != Vector2.ZERO: # if direction is not zero (the player is moving)
		velocity = direction * SPEED # we will move the player around
		last_direction = direction  # and we will store the last direction which will be set as the current direction
	else: # if the direction is zero (the player is not moving)
		velocity = Vector2.ZERO
	
	
	
func process_animation() -> void:
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
```
---

**Here:**

"@onready var animated_sprite_2d: AnimatedSprite2D = $AnimatedSprite2D" is a variable created which gives access to the sprite animations

in physics_process we call for the process_movement and process_animation function

process_movement checks the speed of the player and determines if they're either walking or idle

based off either one of those, the play_animation function will run either "walk" or "idle" and then calls the appropriate animation

---

**important:**
"void" is used when a function is not returning anything

"prefix" is used as a placeholder for either walk or idle, which will depend on the velocity either being equal or not equal to Vector2

## Extra notes

# Collision configuration
In the Godot engine, only collision bodies/objects can collide with each other if they are on the same layer, sort of as a category.

Although, mask can define more specifically *what* a body/object can collide *with*.
