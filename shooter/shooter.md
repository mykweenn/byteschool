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
