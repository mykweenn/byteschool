# CosmoShooter

## Урок 1

### Создание игрока (передвижения)

Начнем разработку нашей игры с создания корабля, на котором мы будем разрушать астеройды

Для его создания нам нужен узел CharacterBody2D и к нему добавить следующие узлы:
* AnimatedSprite2D
* CollisionShape2D (Можно использовать CollisionPolygon2D если у формы очень много углов)
* Node2D 2 узла (Это место вылета снарядов)

### Переходим к работе со скриптом

Сначала зададим перемещение. Для этого создадим 3 переменные

```gdscript
@export var acceleration := 10.0
@export var max_speed := 350.0
@export var rotation_speed := 250.0
```

Задидаим перемещение вперед и назад, а также ограничим скорость перемещения

```gdscript
	var input_vector := Vector2(0,Input.get_axis("up","down"))
	velocity += input_vector.rotated(rotation) * acceleration #перемещение в пространстве
	velocity = velocity.limit_length(max_speed) #макс скорость
```

Следующим шагом добавим поворот нашего корабля

```gdscript
if Input.is_action_pressed("right"): #поворот вправо
		rotate(deg_to_rad(rotation_speed*delta))
	if Input.is_action_pressed("left"): #поворот влево
		rotate(deg_to_rad(-rotation_speed*delta))
```

Теперь наш корабль может двигаться, однако он не перестает это делать в направлении вектора даже если мы отпустили все кнопки. Чтобы это исправить мы добавим еще одно условие 

```gdscript
if input_vector.y == 0: #останавка игрока при отпущенной клавише
		velocity = velocity.move_toward(Vector2.ZERO,3)
```

Так как мы будет ограничены в рамках изначальных размеров поля и у нас не должно быть возможности вылета за экран добавим условия на проверку этого
```gdscript
	var screen_size = get_viewport_rect().size #размер экрана
	if global_position.y <0: #п
		global_position.y = screen_size.y #п
	elif global_position.y > screen_size.y: #п
		global_position.y = 0 #п
	if global_position.x <0: #п
		global_position.x = screen_size.x #п
	elif global_position.x > screen_size.x: #п
		global_position.x = 0 #п
```

### Создание игрока (выстрел)

Начнем создание выстрела с создания самого лазера. Для его создания нам нужен узел Area2D и к нему добавить следующие узлы:

* Sprite2D
* CollisionShape2D
* VisibleOnScreenNotifier2D (Будет удалять снаряды за экраном)

![image](https://github.com/Sindikaty/byteschool/assets/158248099/20430f48-72ca-4f13-849b-d5af0ae41c8d)

### Переходим к работе со скриптом

Сначала зададим перемещение снаряда. Для этого создадим 2 переменные

```gdscript
var movement_vector := Vector2(0,-1)
var speed := 500.0
```

После чего будем задавать глобальную позициию лазера

```gdscript
func _physics_process(delta):
	global_position += movement_vector.rotated(rotation) * speed * delta #поворот лазера в ту сторону куда повернут игрок
	pass
```

Также сразу можно присоединить сигнал screen_exited() для удаления лазера

![image](https://github.com/Sindikaty/byteschool/assets/158248099/fb84b526-6111-4f15-a52b-bea6dde301d8)

И прописать его удаление

![image](https://github.com/Sindikaty/byteschool/assets/158248099/de60acc7-4da4-467e-8af2-1a4e9e325b33)

Теперь можно вернуться к игроку и создать у него функцию shoot_laser(). Для нее нам понадобится 3 переменные

```gdscript
signal laser_shot(laser) # Сигнал и какой параметр будет передаваться
@onready var muzzle = $Muzzle
var laser_scene = preload("res://scenes/laser.tscn")
```

В самой же функции определяем напрявление выстрела и то место откуда он будет произведен, и передаем это значение в сигнал

```gdscript
func shoot_laser():
	var l = laser_scene.instantiate()
	l.global_position = muzzle.global_position # откуда стреляем
	l.rotation = rotation # напрявление куда стреляем
	emit_signal("laser_shot", l)
```

Далее нам понадобится 2 переменные

```gdscript
var shoot_cd = false
var rate_of_fire = 0.35
```

В _process(delta) прописываем выстрел на нажатие кнопки с созданием таймера для создания времени между выстрелами.

```gdscript
func _process(delta):
	if Input.is_action_pressed("shoot"): #стрельба на зажатие клавишы
		if !shoot_cd:
			shoot_cd = true
			shoot_laser()
			await get_tree().create_timer(rate_of_fire).timeout #время перезарядки
			shoot_cd = false
```

Последним что нам осталось для произведение выстрела это вернуться на наш уровень. В нем мы создадим Node2D отвечающий за место выстрела и звук выстрела

![image](https://github.com/Sindikaty/byteschool/assets/158248099/165191bd-95a5-407c-b9eb-47bff21fe170)

Далее создадим скрипт у нашего уровня. В нем создадим 2 переменные

```gdscript
@onready var lasers = $Lasers
@onready var player = $player
```

Создадим новую функцию в которой мы будем включать звук выстрела и добавлять клон нашего лазера

```gdscript
func _on_player_laser_shot(laser):
	$Audio/shoot.play()
	lasers.add_child(laser)
```

И наконец нам осталось подключить сигнал в котором мы хранили координаты и направление куда полетит выстрел к новой функции создания клона нашего лазера

```gdscript
func _ready():
	player.connect("laser_shot", _on_player_laser_shot)
```

Стрельба готова







