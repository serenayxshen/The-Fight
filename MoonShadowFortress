extends Node3D

# ==================================================
# MOONSHADOW FORTRESS - COMBAT PROTOTYPE V0.2
# Group attacks, enemy HP, enemy pursuit and counterattacks
# ==================================================

var player: Node3D
var enemies: Array[Node3D] = []
var camera_3d: Camera3D
var camera_yaw: float = 0.0
var camera_pitch: float = -0.70
var camera_distance: float = 5.0
var hud: Label
var left_saber: MeshInstance3D
var right_saber: MeshInstance3D
var saber_swing_time: float = 0.0
var saber_swing_direction: float = 1.0

var player_hp: float = 100.0
var player_max_hp: float = 100.0
# Shared resources for all heroes
var essence_inventory: int = 0
var gold: int = 0
var essence_drops: Array[Node3D] = []
const ESSENCE_SELL_PRICE: int = 5
var attack_cooldown: float = 0.0
var enemy_attack_cooldown: float = 0.0
var message_time: float = 0.0

var facing: Vector3 = Vector3(0, 0, -1)
var attack_message: String = ""

var floor_mat: StandardMaterial3D
var wall_mat: StandardMaterial3D
var dark_mat: StandardMaterial3D
var crystal_mat: StandardMaterial3D
var player_mat: StandardMaterial3D
var enemy_mat: StandardMaterial3D
var guard_mat: StandardMaterial3D
var elite_mat: StandardMaterial3D



func _ready() -> void:
	print("测试：_ready 已执行")
	_create_materials()
	_create_arena()
	_create_player()
	_create_enemies()
	_create_camera()
	_create_hud()
	_update_hud()

func _process(delta: float) -> void:
	_collect_nearby_essence()
	_update_saber_animation(delta)

	attack_cooldown = maxf(0.0, attack_cooldown - delta)
	enemy_attack_cooldown = maxf(0.0, enemy_attack_cooldown - delta)
	message_time = maxf(0.0, message_time - delta)

	if player_hp > 0.0:
		_move_player(delta)
		_handle_attacks()

	camera_yaw = lerp_angle(
		 camera_yaw,
		player.rotation.y,
		1.0 - exp(-4.0 * delta)
	)

	var camera_offset := Vector3(
		sin(camera_yaw) * cos(camera_pitch),
		-sin(camera_pitch),
		cos(camera_yaw) * cos(camera_pitch)
	) * camera_distance

	camera_3d.global_position = player.global_position + camera_offset
	camera_3d.look_at(
		player.global_position + Vector3(0, 1, 0),
		Vector3.UP)
	
	_update_enemies(delta)
	_update_hud()


func _collect_nearby_essence() -> void:
	for essence in essence_drops.duplicate():
		if not is_instance_valid(essence):
			essence_drops.erase(essence)
			continue

		if player.global_position.distance_to(essence.global_position) <= 2.0:
			var amount: int = essence.get_meta("amount", 1)
			essence_inventory += amount
			essence_drops.erase(essence)
			essence.queue_free()

# ==================================================
# MATERIALS
# ==================================================

func _make_material(color: Color, metallic: float = 0.0) -> StandardMaterial3D:
	var mat := StandardMaterial3D.new()
	mat.albedo_color = color
	mat.metallic = metallic
	mat.roughness = 0.7
	return mat


func _create_materials() -> void:
	floor_mat = _make_material(Color(0.12, 0.13, 0.17))
	wall_mat = _make_material(Color(0.19, 0.17, 0.25))
	dark_mat = _make_material(Color(0.07, 0.06, 0.10))
	crystal_mat = _make_material(Color(0.30, 0.70, 1.0), 0.3)
	crystal_mat.emission_enabled = true
	crystal_mat.emission = Color(0.12, 0.35, 1.0)
	crystal_mat.emission_energy_multiplier = 2.0

	player_mat = _make_material(Color(0.16, 0.13, 0.22), 0.25)
	enemy_mat = _make_material(Color(0.85, 0.62, 0.20), 0.2)
	guard_mat = _make_material(Color(0.60, 0.48, 0.24), 0.3)
	elite_mat = _make_material(Color(0.45, 0.25, 0.75), 0.3)


# ==================================================
# SIMPLE 3D BUILDING HELPERS
# ==================================================

func _add_box(
	parent: Node3D,
	box_size: Vector3,
	box_position: Vector3,
	mat: Material
) -> MeshInstance3D:
	var mesh_instance := MeshInstance3D.new()
	var mesh := BoxMesh.new()
	mesh.size = box_size
	mesh_instance.mesh = mesh
	mesh_instance.position = box_position
	mesh_instance.material_override = mat
	parent.add_child(mesh_instance)
	return mesh_instance


func _add_capsule(
	parent: Node3D,
	radius: float,
	height: float,
	pos: Vector3,
	mat: Material
) -> MeshInstance3D:
	var instance := MeshInstance3D.new()
	var mesh := CapsuleMesh.new()
	mesh.radius = radius
	mesh.height = height
	instance.mesh = mesh
	instance.position = pos
	instance.material_override = mat
	parent.add_child(instance)
	return instance


func _add_sphere(
	parent: Node3D,
	radius: float,
	pos: Vector3,
	mat: Material
) -> MeshInstance3D:
	var instance := MeshInstance3D.new()
	var mesh := SphereMesh.new()
	mesh.radius = radius
	mesh.height = radius * 2.0
	instance.mesh = mesh
	instance.position = pos
	instance.material_override = mat
	parent.add_child(instance)
	return instance


# ==================================================
# MOONSHADOW FORTRESS ARENA
# ==================================================

func _create_arena() -> void:
	var ground := MeshInstance3D.new()
	var ground_mesh := PlaneMesh.new()
	ground_mesh.size = Vector2(38, 38)
	ground.mesh = ground_mesh
	ground.material_override = floor_mat
	ground.position.y = -0.15
	add_child(ground)
	print("测试：地面已创建")

	# Rear wall
	_add_box(self, Vector3(38, 4, 1), Vector3(0, 2, -19), wall_mat)

	# Side walls
	_add_box(self, Vector3(1, 4, 38), Vector3(-19, 2, 0), wall_mat)
	_add_box(self, Vector3(1, 4, 38), Vector3(19, 2, 0), wall_mat)

	# Front wall, leaving the gate entrance
	_add_box(self, Vector3(13, 4, 1), Vector3(-12.5, 2, 19), wall_mat)
	_add_box(self, Vector3(13, 4, 1), Vector3(12.5, 2, 19), wall_mat)

	# Gate towers
	for x_pos in [-6.0, 6.0]:
		_add_box(
			self, Vector3(2, 7, 2),
			Vector3(x_pos, 3.5, 18), dark_mat
		)
		_add_box(
			self, Vector3(0.7, 1.5, 0.7),
			Vector3(x_pos, 6.0, 16.8), crystal_mat
		)

	# Fortress pillars
	for x_pos in [-14.0, -9.0, 9.0, 14.0]:
		_add_box(
			self, Vector3(0.8, 3.5, 0.8),
			Vector3(x_pos, 1.75, -13), dark_mat
		)

	var sun := DirectionalLight3D.new()
	sun.rotation_degrees = Vector3(-55, -25, 0)
	sun.light_energy = 1.4
	add_child(sun)

	var environment_node := WorldEnvironment.new()
	var env := Environment.new()
	env.background_mode = Environment.BG_COLOR
	env.background_color = Color(0.035, 0.025, 0.07)
	env.ambient_light_source = Environment.AMBIENT_SOURCE_COLOR
	env.ambient_light_color = Color(0.38, 0.34, 0.55)
	env.ambient_light_energy = 0.8
	environment_node.environment = env
	add_child(environment_node)


# ==================================================
# PLAYER: KAELEN
# ==================================================

func _create_player() -> void:
	player = Node3D.new()
	player.name = "Kaelen"
	player.position = Vector3(0, 0, 10)
	add_child(player)

	# Body and head
	_add_capsule(player, 0.55, 1.8, Vector3(0, 1.0, 0), player_mat)
	_add_sphere(
		player, 0.38, Vector3(0, 2.05, 0),
		_make_material(Color(0.72, 0.75, 0.83))
	)

	# Silver hair
	_add_sphere(
		player, 0.40, Vector3(0, 2.30, -0.04),
		_make_material(Color(0.78, 0.82, 0.92))
	)

	# Shoulder armor
	var armor_mat := _make_material(Color(0.15, 0.75, 1.0))
	_add_sphere(
		player, 0.32, Vector3(-0.48, 1.65, 0),
		armor_mat
	)
	_add_sphere(
		player, 0.32, Vector3(0.48, 1.65, 0),
		armor_mat
	)

	# Twin sabers
	var blade_mat := _make_material(Color(0.68, 0.83, 1.0), 0.6)
	
	left_saber = _add_box(
		player, Vector3(0.12, 1.0, 0.12),
		Vector3(-0.72, 1.0, -0.15), blade_mat
	)
	right_saber = _add_box(
		player, Vector3(0.12, 1.0, 0.12),
		Vector3(0.72, 1.0, -0.15), blade_mat
	)

func _move_player(delta: float) -> void:
	var turn_input := 0.0

	if Input.is_key_pressed(KEY_A) or Input.is_key_pressed(KEY_LEFT):
		turn_input += 1.0
	if Input.is_key_pressed(KEY_D) or Input.is_key_pressed(KEY_RIGHT):
		turn_input -= 1.0

	# Turn the player
	player.rotation.y += turn_input * 2.5 * delta

	var move_input := 0.0

	if Input.is_key_pressed(KEY_W) or Input.is_key_pressed(KEY_UP):
		move_input += 1.0
	if Input.is_key_pressed(KEY_S) or Input.is_key_pressed(KEY_DOWN):
		move_input -= 1.0

	# Move relative to the player's facing direction
	if absf(move_input) > 0.01:
		var forward := -player.global_transform.basis.z
		player.position += forward * move_input * 7.0 * delta

	# Update facing direction
	facing = -player.global_transform.basis.z

	# Keep the hero inside the arena
	player.position.x = clampf(player.position.x, -17.0, 17.0)
	player.position.z = clampf(player.position.z, -17.0, 17.0)

# ==================================================
# ENEMIES: VOLTARA ARMY
# ==================================================

func _create_enemies() -> void:
	_spawn_enemy("Lightning Blade Soldier", Vector3(-5, 0, -5), 45.0, 2.0, 3.0, enemy_mat)
	_spawn_enemy("Lightning Blade Soldier", Vector3(0, 0, -6), 45.0, 2.0, 3.0, enemy_mat)
	_spawn_enemy("Lightning Blade Soldier", Vector3(5, 0, -5), 45.0, 2.0, 3.0, enemy_mat)
	_spawn_enemy("Lightning Blade Soldier", Vector3(-8, 0, -9), 45.0, 2.0, 3.0, enemy_mat)
	_spawn_enemy("Lightning Blade Soldier", Vector3(8, 0, -9), 45.0, 2.0, 3.0, enemy_mat)

	_spawn_enemy("Lightning Guard", Vector3(-3, 0, -12), 90.0, 3.0, 4.0, guard_mat)
	_spawn_enemy("Lightning Guard", Vector3(3, 0, -12), 90.0, 3.0, 4.0, guard_mat)

	_spawn_enemy("Elite Captain", Vector3(0, 0, -15), 180.0, 4.0, 5.0, elite_mat)


func _spawn_enemy(
	enemy_name: String,
	spawn_pos: Vector3,
	hp: float,
	speed: float,
	damage: float,
	mat: Material
) -> void:
	var enemy := Node3D.new()
	enemy.name = enemy_name
	enemy.position = spawn_pos

	enemy.set_meta("hp", hp)
	enemy.set_meta("max_hp", hp)
		# Determine enemy rank for essence drops
	if enemy_name == "Elite Captain":
		enemy.set_meta("rank", "commander")
	elif enemy_name == "Lightning Guard":
		enemy.set_meta("rank", "elite")
	else:
		enemy.set_meta("rank", "soldier")
	enemy.set_meta("speed", speed)
	enemy.set_meta("damage", damage)
	enemy.set_meta("attack_timer", randf_range(0.5, 1.5))
	enemy.set_meta("alive", true)

	_add_capsule(enemy, 0.48, 1.7, Vector3(0, 0.95, 0), mat)
	_add_sphere(
		enemy, 0.32, Vector3(0, 1.95, 0),
		_make_material(Color(0.87, 0.75, 0.55))
	)

	# A small golden weapon
	_add_box(
		enemy, Vector3(0.12, 0.85, 0.12),
		Vector3(0.58, 1.0, -0.1),
		_make_material(Color(1.0, 0.77, 0.25), 0.4)
	)

	var hp_label := Label3D.new()
	hp_label.name = "HPLabel"
	hp_label.text = str(int(hp)) + " HP"
	hp_label.position = Vector3(0, 2.7, 0)
	hp_label.font_size = 32
	hp_label.pixel_size = 0.008
	hp_label.billboard = BaseMaterial3D.BILLBOARD_ENABLED
	hp_label.modulate = Color(1.0, 0.85, 0.55)
	enemy.add_child(hp_label)

	add_child(enemy)
	enemies.append(enemy)


# ==================================================
# GROUP ATTACKS
# ==================================================

func _handle_attacks() -> void:
	if attack_cooldown > 0.0:
		return

	if Input.is_key_pressed(KEY_J):
		_perform_attack(false)
	elif Input.is_key_pressed(KEY_K):
		_perform_attack(true)


func _perform_attack(heavy: bool) -> void:
	var attack_range: float
	var attack_damage: float
	var cooldown: float
	var attack_name: String

	if heavy:
		attack_range = 6.0
		attack_damage = 38.0
		cooldown = 0.10
		attack_name = "HEAVY ATTACK!"
	else:
		attack_range = 4.2
		attack_damage = 18.0
		cooldown = 0.38
		attack_name = "QUICK ATTACK!"

	attack_cooldown = cooldown
	saber_swing_time = 0.25
	saber_swing_direction = 1.0 if not heavy else -1.0

	var hit_count := 0

	for enemy in enemies:
		if not is_instance_valid(enemy):
			continue
		if not bool(enemy.get_meta("alive", false)):
			continue

		var to_enemy: Vector3 = enemy.global_position - player.global_position
		var distance := Vector2(to_enemy.x, to_enemy.z).length()

		if distance > attack_range:
			continue

		# Attacks focus on the direction Kaelen is facing.
		var direction_to_enemy := Vector3(to_enemy.x, 0, to_enemy.z).normalized()
		var facing_dot := facing.dot(direction_to_enemy)

		if facing_dot < -0.15:
			continue

		_damage_enemy(enemy, attack_damage)
		hit_count += 1

	if hit_count > 0:
		attack_message = attack_name + "  HIT " + str(hit_count) + " ENEMIES!"
	else:
		attack_message = attack_name + "  MISS!"

	message_time = 1.0
	
	
func _drop_essence(enemy: Node3D) -> void:
	var rank: String = enemy.get_meta("rank", "soldier")
	var amount: int = 0

	if rank == "commander":
		amount = 4
	elif rank == "elite":
		amount = 2
	else:
		if randf() < 0.35:
			amount = 1

	if amount <= 0:
		return

	var essence := MeshInstance3D.new()
	var crystal := SphereMesh.new()
	crystal.radius = 0.25
	crystal.height = 0.5
	essence.mesh = crystal

	var material := StandardMaterial3D.new()
	material.albedo_color = Color(0.2, 1.0, 0.8)
	material.emission_enabled = true
	material.emission = Color(0.1, 1.0, 0.7)
	essence.material_override = material

	essence.position = enemy.global_position + Vector3(0, 0.5, 0)
	essence.set_meta("amount", amount)
	add_child(essence)
	essence_drops.append(essence)

func _damage_enemy(enemy: Node3D, damage: float) -> void:
	var hp: float = float(enemy.get_meta("hp"))
	hp -= damage
	enemy.set_meta("hp", hp)

	var hp_label := enemy.get_node_or_null("HPLabel") as Label3D
	if hp_label:
		hp_label.text = str(maxi(0, int(ceil(hp)))) + " HP"

	# Brief hit flash
	var body := enemy.get_child(0) as MeshInstance3D
	if body:
		body.material_override = _make_material(Color(1.0, 0.95, 0.75))

	if hp <= 0.0:
		enemy.set_meta("alive", false)
		enemy.set_meta("dying", true)
		enemy.set_meta("death_timer", 2.5)
		enemy.set_meta("fall_direction", -1.0 if randf() < 0.5 else 1.0)
		enemies.erase(enemy)


# ==================================================
# ENEMY PURSUIT AND COUNTERATTACKS
# ==================================================

func _update_enemies(delta: float) -> void:
	if player_hp <= 0.0:
		return

	for enemy in enemies:
		if not is_instance_valid(enemy):
			continue
		if not bool(enemy.get_meta("alive", false)):
			continue

		var offset: Vector3 = player.global_position - enemy.global_position
		var distance := Vector2(offset.x, offset.z).length()

		if distance > 1.8:
			var direction := Vector3(offset.x, 0, offset.z).normalized()
			var speed: float = float(enemy.get_meta("speed"))
			enemy.position += direction * speed * delta
		else:
			var timer: float = float(enemy.get_meta("attack_timer"))
			timer -= delta

			if timer <= 0.0:
				var damage: float = float(enemy.get_meta("damage"))
				player_hp = maxf(0.0, player_hp - damage)
				timer = 1.4
				attack_message = "YOU WERE HIT! -" + str(int(damage)) + " HP"
				message_time = 0.8

			enemy.set_meta("attack_timer", timer)


# ==================================================
# CAMERA AND HUD
# ==================================================

func _create_camera() -> void:
	camera_3d = Camera3D.new()
	camera_3d.position = Vector3(0, 4, 6)
	camera_3d.look_at(Vector3(0, 0, 0), Vector3.UP)
	camera_3d.size = 45.0
	add_child(camera_3d)
	camera_3d.look_at(Vector3(0, 0, 0), Vector3.UP)
	camera_3d.current = true


func _create_hud() -> void:
	var canvas := CanvasLayer.new()
	add_child(canvas)

	hud = Label.new()
	hud.position = Vector2(20, 18)
	hud.add_theme_font_size_override("font_size", 20)
	hud.add_theme_color_override("font_color", Color(0.92, 0.91, 1.0))
	canvas.add_child(hud)


func _update_hud() -> void:
	if not is_instance_valid(hud):
		return

	var alive_count := 0
	for enemy in enemies:
		if is_instance_valid(enemy) and bool(enemy.get_meta("alive", false)):
			alive_count += 1

	var status := "MOONSHADOW FORTRESS\n"
	status += "KAELEN HP: " + str(int(player_hp)) + " / " + str(int(player_max_hp)) + "\n"
	status += "VOLTARA ENEMIES: " + str(alive_count) + "\n"
	status += "ESSENCE: " + str(essence_inventory) + "\n"
	status += "GOLD: " + str(gold) + "\n"
	status += "WASD / ARROWS: MOVE\n"
	status += "J: QUICK ATTACK     K: HEAVY ATTACK\n"

	if player_hp <= 0.0:
		status += "\nDEFEATED - RESTART THE SCENE TO TRY AGAIN"
	elif message_time > 0.0:
		status += "\n" + attack_message

	hud.text = status

func _update_saber_animation(delta: float) -> void:
	if saber_swing_time > 0.0:
		saber_swing_time = maxf(0.0, saber_swing_time - delta)

		var swing_progress: float = 1.0 - saber_swing_time / 0.25
		var swing_angle: float = sin(swing_progress * PI) * 1.2 * saber_swing_direction

		left_saber.rotation.z = swing_angle
		right_saber.rotation.z = -swing_angle
	else:
		left_saber.rotation.z = 0.0
		right_saber.rotation.z = 0.0

func _unhandled_input(event: InputEvent) -> void:
	if event is InputEventMouseButton:
		if event.button_index == MOUSE_BUTTON_RIGHT:
			Input.mouse_mode = Input.MOUSE_MODE_CAPTURED if event.pressed else Input.MOUSE_MODE_VISIBLE

	elif event is InputEventMouseMotion:
		if Input.mouse_mode == Input.MOUSE_MODE_CAPTURED:
			camera_yaw -= event.relative.x * 0.005
			camera_pitch -= event.relative.y * 0.005
			camera_pitch = clampf(camera_pitch, -0.8, 0.5)
