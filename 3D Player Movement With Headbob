extends CharacterBody3D

#Player Collision Nodes
@onready var player_standing_col: CollisionShape3D = $Player_Standing_Col
@onready var player_crouching_col: CollisionShape3D = $Player_Crouching_Col

#Player RayCast Nodes
@onready var crouch_check: RayCast3D = $Crouch_Check

#Player Camera Nodes
@onready var head = $Camera_Handler/Head
@onready var head_bob_controller: Node3D = $Camera_Handler
@onready var camera = $Camera_Handler/Head/Camera_3D

#Player State Display Nodes
@onready var states_label: Label3D = $States_Label

#Mouse Sensitivity
var sens = 0.002

#Gravity Strength
var gravity = 40.0

#Speed Variables
var run_speed = 20.0
var walk_speed = 12.0
var crouch_speed = 7.0
var air_control = 6.0
var speed = 12.0
var lerp_speed = 10.0
var air_lerp_speed = 1.0

#Jump Strength
var jump_velocity = 18.0

#Player Camera Height
const STANDING_HEIGHT = 1.6
const CROUCHING_HEIGHT = 0.6

#Player Direction
var direction = Vector3.ZERO

#Player State Variables
var running = false
var walking = false
var crouching = false
var idle = true
var in_air = false

#Headbob Values
const HEAD_BOBBING_RUNNING_SPEED = 22.0
const HEAD_BOBBING_WALKING_SPEED = 12.0
const HEAD_BOBBING_CROUCHING_SPEED = 9.0
const HEAD_BOBBING_IDLE_SPEED = 2.0

const HEAD_BOBBING_RUNNING_INTENSITY = 0.2
const HEAD_BOBBING_WALKING_INTENSITY = 0.1
const HEAD_BOBBING_CROUCHING_INTENSITY = 0.05
const HEAD_BOBBING_IDLE_INTENSITY = 0.05

var head_bobbing_vector = Vector2.ZERO
var head_bobbing_index = 0.0
var head_bobbing_current_intensity = 0.0


func _ready() -> void:
	#Capturing the mouse when the scene is loaded.
	Input.mouse_mode = Input.MOUSE_MODE_CAPTURED




func _physics_process(delta: float) -> void:
	#Getting player input.
	var input_dir := Input.get_vector("left", "right", "forward", "backward")


	#Converting player input into a direction.
	if !in_air:
		direction = lerp(direction, (transform.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized(), delta * lerp_speed)
	else:
		direction = lerp(direction, (transform.basis * Vector3(input_dir.x, 0, input_dir.y)).normalized(), delta * air_lerp_speed)


	#Gravity logic.
	if not is_on_floor():
		velocity.y -= gravity * delta
		in_air = true
		states_label.text = "in air"
	else:
		in_air = false


	#Mouse escape logic.
	if Input.is_action_just_pressed("escape"):
		Input.mouse_mode = Input.MOUSE_MODE_VISIBLE


	#Jumping logic.
	if Input.is_action_just_pressed("jump") and is_on_floor():
		velocity.y = jump_velocity


	#Idle logic.
	if !input_dir and !in_air:
		running = false
		walking = false
		idle = true
	else:
		idle = false

	#Walking logic.
	if input_dir and !in_air and !running and !crouching:
		running = false
		walking = true
		crouching = false
		idle = false
	else:
		walking = false


	#Crouching state logic.
	if Input.is_action_pressed("crouch") and !in_air:
		running = false
		walking = false
		crouching = true
	elif !crouch_check.is_colliding():
		player_crouching_col.disabled = true
		player_standing_col.disabled = false
		head.position.y = lerp(head.position.y, STANDING_HEIGHT, delta * lerp_speed)
		speed = lerp(speed, walk_speed, delta * lerp_speed)
		crouching = false


	#Running state logic.
	if Input.is_action_pressed("run") and !in_air and !crouching and input_dir:
		running = true
		walking = false
		crouching = false
		idle = false
	else:
		running = false


	#Running logic.
	if running:
		states_label.text = "running"
		speed = lerp(speed, run_speed, delta * lerp_speed)
		head_bobbing_current_intensity = HEAD_BOBBING_RUNNING_INTENSITY
		head_bobbing_index += HEAD_BOBBING_RUNNING_SPEED * delta


	#Walking logic.
	elif walking:
		states_label.text = "walking"
		head_bobbing_current_intensity = HEAD_BOBBING_WALKING_INTENSITY
		head_bobbing_index += HEAD_BOBBING_WALKING_SPEED * delta


	#Crouching logic.
	elif crouching:
		states_label.text = "crouching"
		player_crouching_col.disabled = false
		player_standing_col.disabled = true
		head.position.y = lerp(head.position.y, CROUCHING_HEIGHT, delta * lerp_speed)
		speed = lerp(speed, crouch_speed, delta * lerp_speed)
		head_bobbing_current_intensity = HEAD_BOBBING_CROUCHING_INTENSITY
		head_bobbing_index += HEAD_BOBBING_CROUCHING_SPEED * delta
		if idle:
			states_label.text = "crouching and idle"


	#Idle logic.
	elif idle:
		states_label.text = "idle"
		head_bobbing_current_intensity = HEAD_BOBBING_IDLE_INTENSITY
		head_bobbing_index += HEAD_BOBBING_IDLE_SPEED * delta


	#Head bobbing logic.
	if !in_air and input_dir != Vector2.ZERO:
		head_bobbing_vector.y = sin(head_bobbing_index)
		head_bobbing_vector.x = sin(head_bobbing_index/2)+0.5
		head_bob_controller.position.y = lerp(head_bob_controller.position.y, head_bobbing_vector.y * (head_bobbing_current_intensity/2.0), delta * lerp_speed)
		head_bob_controller.position.x = lerp(head_bob_controller.position.x, head_bobbing_vector.x * head_bobbing_current_intensity, delta * lerp_speed)
	elif !in_air and input_dir == Vector2.ZERO:
		head_bobbing_vector.y = sin(head_bobbing_index)
		head_bob_controller.position.y = lerp(head_bob_controller.position.y, head_bobbing_vector.y * (head_bobbing_current_intensity/2.0), delta * lerp_speed)
	else:
		head_bob_controller.position.y = lerp(head_bob_controller.position.y, 0.0, delta * lerp_speed)
		head_bob_controller.position.x = lerp(head_bob_controller.position.x, 0.0, delta * lerp_speed)


	#Velocity logic.
	if direction:
		velocity.x = direction.x * speed
		velocity.z = direction.z * speed
	else:
		velocity.x = move_toward(velocity.x, 0, speed)
		velocity.y = move_toward(velocity.y, 0, speed)
	
	#Allowing the player to move
	move_and_slide()




func _input(event: InputEvent) -> void:
	#Rotating the camera using the mouse position.
	if event is InputEventMouseMotion:
		rotate_y(-event.relative.x * sens)
		camera.rotate_x(-event.relative.y * sens)
		camera.rotation.x = clamp(camera.rotation.x, deg_to_rad(-90), deg_to_rad(90))
