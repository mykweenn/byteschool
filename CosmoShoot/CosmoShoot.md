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
func _physics_process(delta):
	var input_vector := Vector2(0,Input.get_axis("up","down"))
	velocity += input_vector.rotated(rotation) * acceleration #перемещение в пространстве
	velocity = velocity.limit_length(max_speed) #макс скорость
	...
```

Следующим шагом добавим поворот нашего корабля

```gdscript
func _physics_process(delta):
	...
	if Input.is_action_pressed("right"): #поворот вправо
		rotate(deg_to_rad(rotation_speed*delta))
	if Input.is_action_pressed("left"): #поворот влево
		rotate(deg_to_rad(-rotation_speed*delta))
	...
```

Теперь наш корабль может двигаться, однако он не перестает это делать в направлении вектора даже если мы отпустили все кнопки. Чтобы это исправить мы добавим еще одно условие 

```gdscript
func _physics_process(delta):
	...
if input_vector.y == 0: #останавка игрока при отпущенной клавише
		velocity = velocity.move_toward(Vector2.ZERO,3)
	...
```

Так как мы будет ограничены в рамках изначальных размеров поля и у нас не должно быть возможности вылета за экран добавим условия на проверку этого
```gdscript
func _physics_process(delta):
	...
	var screen_size = get_viewport_rect().size #размер экрана
	if global_position.y <0: #п
		global_position.y = screen_size.y #п
	elif global_position.y > screen_size.y: #п
		global_position.y = 0 #п
	if global_position.x <0: #п
		global_position.x = screen_size.x #п
	elif global_position.x > screen_size.x: #п
		global_position.x = 0 #п
	...
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

Теперь можно вернуться к игроку и создать у него функцию shoot_laser(). Для нее нам понадобится 2 переменные и сигнал

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

### Создание астероидов

Для астероидов нам понадобится узел Area2D и к нему добавить следующие узлы:

* Sprite2D
* CollisionShape2D

![image](https://github.com/Sindikaty/byteschool/assets/158248099/b3fa5450-0f01-40b2-9fea-81e6473770a7)

Однако астероидов у нас 3 вида, а создаем мы только 1, так что же нам делать? Со спрайтами все просто, они у нас уже есть, а с коллизиями мы поступим следующим образом. Начнем создание астеройдов с самого маленького и после создания сохраняем коллизию в отдельный файл, и переходим к следующему, а когда доходим до большого также сохраняем и оставляем его

![image](https://github.com/Sindikaty/byteschool/assets/158248099/304e41d4-5ae7-4566-b7e9-47828a2d53d3)

![image](https://github.com/Sindikaty/byteschool/assets/158248099/8216241d-9707-4c85-b5d7-896efacf40c0)

После создания астероидов начнем создавать переменные которые нам будут нужны в дальнейшем, например, переменную определяющую размер астероида, размер очков за его уничтожение и т.д.

```gdscript
@onready var sprite = $Sprite2D
@onready var cshape = $CollisionShape2D

enum AsteroidSize{LARGE,MEDIUM,SMALL} # перечисления (enum)
@export var size := AsteroidSize.LARGE

var movement_vector := Vector2(0,-1)
var speed := 50

var points: int:
	get:
		match size:
			AsteroidSize.LARGE:
				return 100
			AsteroidSize.MEDIUM:
				return 50
			AsteroidSize.SMALL:
				return 25
			_:
				return 0
```

Добавим перемещение нашим астероидам. Начнем с функции _ready(), чтобы задать изначальные параметры движения (чему равна скорости всех астероидов, задать их текстуры и коллизии)
```gdscript
func _ready():
    rotation = randf_range(0, 2*PI) # Устанавливаем случайный угол поворота для астероида
    match size: # Используем конструкцию match для установки параметров в зависимости от размера астероида
        AsteroidSize.LARGE:
            speed = randf_range(50,100) # Устанавливаем случайную скорость для большого астероида
            sprite.texture = preload("res://textures/B_meteor.png") # Устанавливаем текстуру для большого астероида
            cshape.set_deferred("shape", preload("res://resourse/asteroid_b.tres")) # Устанавливаем форму столкновений для большого астероида
        AsteroidSize.MEDIUM:
            speed = randf_range(100,150) # Устанавливаем случайную скорость для среднего астероида
            sprite.texture = preload("res://textures/M_meteor.png") # Устанавливаем текстуру для среднего астероида
            cshape.set_deferred("shape", preload("res://resourse/asteroid_m.tres")) # Устанавливаем форму столкновений для среднего астероида
        AsteroidSize.SMALL:
            speed = randf_range(100,200)  # Устанавливаем случайную скорость для маленького астероида
            sprite.texture = preload("res://textures/S_meteor.png") # Устанавливаем текстуру для маленького астероида
            cshape.set_deferred("shape", preload("res://resourse/asteroid_s.tres")) # Устанавливаем форму столкновений для маленького астероида
```

Зададим передвижение астеройдам, а также добавим ограничение на вылет за экран и им 

```gdscript
func _physics_process(delta):
	global_position += movement_vector.rotated(rotation) * speed * delta
	
	var radius = cshape.shape.radius
	
	var screen_size = get_viewport_rect().size #перемещение в начало/конец астероида
	if (global_position.y + radius) < 0: #п
		global_position.y = (screen_size.y + radius) #п
	elif (global_position.y - radius) > screen_size.y: #п
		global_position.y = -radius #п
	if (global_position.x + radius) < 0: #п
		global_position.x = (screen_size.x + radius) #п
	elif (global_position.x - radius) > screen_size.x: #п
		global_position.x =  -radius #п
```

Также в дальнейшем для удобства обращение к переменным у астероида добавим имя класса у нашей Area2D

```gdscript
class_name Asteroid extends Area2D
```

Теперь наши астеройды двигаются, но мы никак не можем с ними взаимодейтвовать.

## Урок 2

Начнем создавать уничтожение астероидов. Для этого нам понадобится сигнал который будет передавать уже 3 параметра

```gdscript
signal exploded(pos, size, points)
```

И создаем функцию в которой этот сигнал получает информацию об астероиде

```gdscript
func explode():
	emit_signal("exploded", global_position, size, points)
	queue_free()
```

И теперь нам нужно присоединить сигнал area_entered() у `лазера`

```gdscript
func _on_area_entered(area):
	if area is Asteroid:
		var asteroid = area
		asteroid.explode()
		queue_free()
```

Теперь наши астероиды уничтожаются, однако уничтожается только большие и ничего больше не происходит. Сделаем чтобы наш большой распадался на 2 средний, а средний на 2 маленьких астероида. Для этого вернемся в скрипт основного уровня и в ней создадим переменную подгружающую нашу сцену с астероидом

```gdscript
var asteroid_scene = preload("res://scenes/asteroid.tscn")
@onready var asteroids = $Asteroids # это узел в котором лежат наши астероиды
```

Теперь нам нужно создать еще 2 функции. Первая функция будет отвечать за спавн астероидов
```gdscript
func spawn_asteroid(pos, size):
    var a = asteroid_scene.instantiate()  # Создаем экземпляр астероида из сцен
    a.global_position = pos # Устанавливаем глобальную позицию для астероида
    a.size = size # Устанавливаем размер для астероида
    a.connect("exploded", _on_asteroid_exploded) # Подключаем сигнал "exploded" к методу _on_asteroid_exploded
    asteroids.call_deferred("add_child", a)  # Добавляем созданный астероид в группу asteroids через вызов метода add_child
```
А второй будет функция отвечающая за то, какой астероид и сколько будет появлятся при уничтожении 
```gdscript
func _on_asteroid_exploded(pos, size, points):
    $AsteroidHitSound.play() # Воспроизводим звук столкновения с астероидом
    for i in range(2): # Создаем два новых астероида после разрушения текущего
        match size:  # Используем конструкцию match для определения размера астероида и создания новых астероидов
            Asteroid.AsteroidSize.LARGE:
                spawn_asteroid(pos, Asteroid.AsteroidSize.MEDIUM) # Если астероид был большим, создаем два средних астероида
            Asteroid.AsteroidSize.MEDIUM:
                spawn_asteroid(pos, Asteroid.AsteroidSize.SMALL)   # Если астероид был средним, создаем два маленьких астероида
            Asteroid.AsteroidSize.SMALL:
```

И последним что осталось это подключить сигнал в _ready()

```gdscript
func _ready():
	...
	for asteroid in asteroids.get_children(): # asteroids это узел в котором лежат наши астероиды
		asteroid.connect("exploded", _on_asteroid_exploded)
```
