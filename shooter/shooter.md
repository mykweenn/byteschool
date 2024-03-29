# 2D-Шутер
Проект, направленный на отработку тем, которые ученик проходил на предыдущих уроках.
В проекте затрагиваются концепции ООП.

## Урок 1
### Создание игрока
Концептуально для создания игрока потребуются следующие узлы:
* CharacterBody2D - Главный узел, для реализации физики персонажа
* AnimationPlayer - Для создания анимации игроку (покраснение при получении урона)
* AnimatedSprite2D - Для создания спрайтов анимации игроку
* CollisionShape2D - Для создания коллизии игроку
* Camera2D - В этом проекте мы будем больше работать с камерой, она нужна не только, чтобы показать где игрок, но и позволит работать с обзором
* Camera2D/Timer - В основном нужен для того, чтобы остановить тряску камеры при выстреле
* AudioStreamPlayer2D (sound_gun) - Нужен для звука с двухмерным позиционированием
* Sprite (gun) - Узел со спрайтом оружия. Из него вызывается пуля
* Marker2D (scope) - Нужен для создания позиции прицела, когда игрок нажмет ПКМ (камера смещается на позицию прицела)
<br>

![image](https://github.com/mykweenn/byteschool/assets/98867083/734bcf6d-286d-49e7-8d13-45ed31aff1f8)


### Переходим к работе со скриптом
Ниже представлены переменные, необходимые для создания передвижения игрока
```gdscript
#movement
@export var speed: int = 100
var input_vector = Vector2.ZERO
```
Ниже пойдет скрипт, необходимый для осуществления передвижения героя
> [!CAUTION]
> Методы написания скрипта может отличаться в зависимости от текстур, которые вы используете. Некоторые текстуры может быть необходимо вращать, а некоторые - нет. Будьте внимательны и готовы к такому

#### Создание передвижения
Давай разберем построчно код ниже. Пояснение написано в комментариях к коду

```gdscript
func _physics_process(delta: float):
	var mouse_pos = get_global_mouse_position() # - получаем позицию мыши
	look_at(mouse_pos) # - персонаж смотрит на курсор (мышь)
	Input.set_custom_mouse_cursor(cursor, Input.CURSOR_ARROW, Vector2(16,16)) # смотри это ниже
	$AnimatedSprite2D.global_rotation = 0.0 # - блокирует возможность вращения текстуры персонажа (если в этом есть необходимость)
```
Код ниже вы уже разбирали на предыдущих уроках (продолжение верхнего кода)

```gdscript
	scope() # зачем это нужно смотри ниже
	get_input_velocity(delta) # зачем это нужно смотри ниже
	input_vector.x = Input.get_action_strength("right") - Input.get_action_strength("left")
	input_vector.y = Input.get_action_strength("down") - Input.get_action_strength("up")
	
	velocity = input_vector * speed

	if(input_vector.x > 0):
		$AnimatedSprite2D.play("walk")
		$AnimatedSprite2D.flip_h = false
	elif(input_vector.x < 0):
		$AnimatedSprite2D.flip_h = true
		$AnimatedSprite2D.play("walk")
	move_and_slide()
```

И добавим прицел игроку 
```gdscript
var cursor = preload("res://textures/white_crosshair.png")

func _physics_process(delta: float):
	Input.set_custom_mouse_cursor(cursor, Input.CURSOR_ARROW, Vector2(16,16))
```
>[!IMPORTANT]
> Обрати внимание на путь, где лежит текстура. У вас может немного отличаться, но лучше все разбивать на отдельные папки


#### Создание деша
Фрагмент кода ниже нужен для создания деша (ускорения)

![](https://github.com/mykweenn/byteschool/blob/main/shooter/img/tumblr_nemrsjk8hi1sulisxo1_1280.webp)

> У нас будет не так красиво выглядеть деш, но по механике будет так работать

```gdscript
func dash(delta):
	transform.origin = lerp(transform.origin, transform.origin + 40 * input_vector , 0.1)
```



Этот код означает следующее:
- transform.origin - это позиция объекта в пространстве.
- lerp() - это функция, которая выполняет линейную интерполяцию между двумя значениями. В данном случае, она применяется к текущей позиции объекта и новой позиции, которая получается при добавлении вектора управления (input_vector), умноженного на 40 .
- 0.1 - это коэффициент сглаживания, который определяет, насколько быстро объект будет перемещаться к новой позиции.

Таким образом, этот код перемещает объект с текущей позиции к новой позиции, которая получается при добавлении вектора управления, умноженного на 40, с использованием линейной интерполяции для плавного перемещения.

Ниже представлен метод, в котором написаны условия для применения деша

```gdscript
func get_input_velocity(delta): # В аргумент
	if Input.is_action_just_pressed("dash"): # Если нажата кнопка деша, то...
		dash_trigger = true # срабатывает триггер деша
	if dash_trigger == true: # если триггер деша равен истине, то
		dash_time += delta # к дешу прибавляется delta (таким образом создается локальный таймер)
		dash(delta) (вызываем функцию деша, которую вы писали ранее)
		if dash_time > 0.25: # Если время деша превышает 0.25 секунды, то
			dash_trigger = false # вырубаем деш
			dash_time = 0 # время деша откатывается к 0
```

Этот код на языке GDScript означает следующее:
- func get_input_velocity(delta): - это объявление функции с именем get_input_velocity, которая принимает аргумент delta.
- if Input.is_action_just_pressed("dash"): dash_trigger = true - если действие "dash" было только что нажато, устанавливается флаг dash_trigger в значение true.
- if dash_trigger == true: - если флаг dash_trigger равен true, то выполняются следующие действия:
  - dash_time += delta - увеличивается переменная dash_time на значение delta.
  - dash(delta) - вызывается функция dash с аргументом delta.
  - if dash_time > 0.25: - если значение переменной dash_time больше 0.25, то выполняются следующие действия:
    - dash_trigger = false - флаг dash_trigger устанавливается в значение false.
    - dash_time = 0 - переменная dash_time устанавливается в значение 0.

Таким образом, этот код отслеживает нажатие кнопки "dash", запускает действие "dash" при установленном флаге dash_trigger, и останавливает действие "dash" после прошествия 0.25 секунд.
> Аргумент delta в функции get_input_velocity(delta) используется для управления временными интервалами и обеспечения плавного движения объекта независимо от фреймрейта (количества кадров в секунду). 
Когда игра работает на компьютере с высокой производительностью, количество кадров в секунду может быть очень высоким, что может привести к тому, что объекты будут двигаться слишком быстро. Использование аргумента delta позволяет умножать скорость объекта на это значение, чтобы обеспечить плавное движение независимо от производительности устройства.
Поэтому аргумент delta помогает сделать игру более плавной и управляемой на разных устройствах с разной производительностью.


#### Создание прицеливания

```gdscript
func scope():
	if Input.is_action_pressed("aim"): # При зажатии ПКМ срабатывает одно из условий ниже
		$Camera2D.position = $scope.position # Если нажат ПКМ, то камера перейдет к позиции прицела
	else:
		$Camera2D.position = Vector2(0,0) # Иначе камера сместится обратно в начальную позицию
```
## Урок 2

### Создание оружия
![image](https://github.com/mykweenn/byteschool/assets/98867083/9da84bea-30aa-41f2-89e8-2bc25a873ab4)

Базово для создания пушки потребуются следующие узлы:
* Sprite2D - Главный узел, является узлом для спрайтов
* Marker2D - Необходим для редактирования двухмерного пространства. Задать точку спавна, вылет пули и т.д.
* AudioStreamPlayer2D - Нужен для позиционированного звукогового сопровождения

Полный скрипт пули, чтобы ты полностью с ним ознакомился:
``` gdscript
extends Sprite2D

var can_fire = true
var bullet = preload("res://bullet.tscn")
var bullet_speed = 1000

var state = "shoot"

var ammoInMagazine = 30
var maxAmmoInMagazine = 30

func _physics_process(delta):
	slowmo()
	var mouse_pos = get_global_mouse_position()
	look_at(mouse_pos)
	if global_rotation_degrees > 90 or global_rotation_degrees < -90:
		self.flip_v = true
	else:
		self.flip_v = false
		
	


func shoot():
	if can_fire and ammoInMagazine > 0:
		var bullet_instance = bullet.instantiate()
		$bullet_point.position = Vector2(30, 0)
		bullet_instance.rotation = rotation + randf_range(-0.1, 0.1)
		bullet_instance.global_position = $bullet_point.global_position
		bullet_instance.apply_impulse(Vector2(bullet_speed, 0).rotated(get_parent().rotation), Vector2())
		Global.camera.shake(0.2, 1)
		get_parent().get_parent().add_child(bullet_instance)
		$sound_gun.play()
		ammoInMagazine -= 1
		can_fire = false
		await get_tree().create_timer(0.1).timeout # Генератор таймера
		can_fire = true

func slowmo():
	if Global.slowmo == true:
		$sound_gun.pitch_scale = 0.8
	else:
		$sound_gun.pitch_scale = 1
		


```
А теперь давай разбирать построчно, поэтапно.
``` gdscript
var can_fire = true # переменная, которая проверяет можно ли стрелять игроку
var bullet = preload("res://bullet.tscn") # путь до сцены с пулей (может отличаться у учеников)
var bullet_speed = 1000 # скорость пули


var ammoInMagazine = 30 # переменная с текущим количеством патрон в магазине
var maxAmmoInMagazine = 30 # переменная с максимальным количеством патрон, которое допустимо в магазине
```
Дальше я убрал пока что вызываемый метод slowmo() из скрипта, так как на данном этапе он пока не нужен

``` gdscript
func _physics_process(delta):
	var mouse_pos = get_global_mouse_position() # переменная куда сохраняется глобальная позиция мыши
	look_at(mouse_pos) # метод который поворачивает текущий узел в сторону указанного объекта
	if global_rotation_degrees > 90 or global_rotation_degrees < -90: # условие при котором если глобальные градусы поворота спрайта превышают 90 или меньше -90
		self.flip_v = true # если условие выше является истиной, то спрайт зеркалится
	else: # иначе
		self.flip_v = false # не зеркалится
```

Здесь мы разберем скрипт для стрельбы
``` gdscript
func shoot():
	if can_fire and ammoInMagazine > 0: # если игрок может стрелять и патрон больше 0
		var bullet_instance = bullet.instantiate() # инстанцируется пуля (создается экземпляр указанной сцены, клон проще говоря)
		$bullet_point.position = Vector2(30, 0) # позиция Marker2D (точка вылета пули) мы ставим в позицию которая находится примерно около рук персонажа
		bullet_instance.rotation = rotation + randf_range(-0.1, 0.1) # инстанцированной пуле мы задаем параметр вращения, добавляя небольшой разброс
		bullet_instance.global_position = $bullet_point.global_position # глобальная позиция инстанцированной пули и точки вылета пули совпадают
		bullet_instance.apply_impulse(Vector2(bullet_speed, 0).rotated(get_parent().rotation), Vector2()) # задаем импульс для инстацнированной пули со скоростью пули и методом указываем направление родительского узла, а вторым вектором - ничего
		Global.camera.shake(0.2, 1) # глобальные настройки тряски камеры
		get_parent().get_parent().add_child(bullet_instance) # получаем родительский узел дважды и добавляем в сцену нового ребенка, который является инстанцированной пулей
		$sound_gun.play() # проигрывание звука пули
		ammoInMagazine -= 1 # отнимаем пульку
		can_fire = false # запрещаем стрелять игроку
		await get_tree().create_timer(0.1).timeout # Генератор таймера, который истечет за указанное время в скобках (проще говоря это у нас скорострельность пушки)
		can_fire = true # разрешаем пальбу
```
