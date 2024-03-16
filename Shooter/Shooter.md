# 2D-Шутер
Проект, направленный на отработку тем, которые ученик проходил на предыдущих уроках. В проекте затрагиваются концепции ООП.

## Урок 1

## Создание карты

Для удобства дальнейшего создания игрока, создадим карту с физическими слоями, но пока что без слоев навигации для ботов. Для этого создаем `TileMap`. В свойствах `TileMap` создаем `TileSet` после чего у нас появится сетка игрового поля. <br>
Добавляем наш набор тайлов, он должен выглядить примерно так:

![image](https://github.com/Sindikaty/byteschool/assets/158248099/ecf9a4ff-8dc5-4ae1-939a-4c10b3b9b515)

Для создания физического слоя нажимаем по `TileSet` и в его свойстве `Physics Layers` добавляем новый физический слой
![image](https://github.com/Sindikaty/byteschool/assets/158248099/2f678e59-d97c-421b-ad38-00050d068b6d)

Возвращаемя в набору тайлов, заходим в рисовать и выбираем там `Физический слов 0`
![image](https://github.com/Sindikaty/byteschool/assets/158248099/c109ea79-8b69-4f0f-94b7-b92e29e7d3b1)

Осталось лишь выделить те ячейки которые мы хотели бы сделать физическими
![image](https://github.com/Sindikaty/byteschool/assets/158248099/8e626731-e2ee-4e05-addb-c81260a2d45d)

Теперь даем волю фантазии и рисуем карту по которой будет перемещаться игрок

### Создание игрока

Для начала создадим CharacterBody2D и доабвим к нему следующие узлы:
* AnimatedSprite2D
* CollisionShape2D
* Camera2D

Добавим анимацию игроку
![image](https://github.com/Sindikaty/byteschool/assets/158248099/c6f65643-f853-440c-89d2-f967d14b4dd1)

### Переходим к работе со скриптом

Сначала зададим перемещение. Для этого создадим 2 переменные отвечающие за скорость и вектор движения

```gdscript
@export var speed : int = 100
var input_vector = Vector2.ZERO
```

После чего зададим само пермещение, а также проигрыванеи анимаций

```gdscript
func _physics_process(delta):
  input_vector.x = Input.get_action_strength("right") - Input.get_action_strength("left")
	input_vector.y = Input.get_action_strength("down") - Input.get_action_strength("up")
	
	velocity = input_vector * speed

	if(input_vector.x > 0):
		$AnimatedSprite2D.play("run")
		$AnimatedSprite2D.flip_h = false
	elif(input_vector.x < 0):
		$AnimatedSprite2D.flip_h = true
		$AnimatedSprite2D.play("run")
	move_and_slide()
```
Ниже пойдет скрипт, необходимый для осуществления поворота персонажа

```gdscript
func _physics_process(delta):
        ...
	var mouse_pos = get_global_mouse_position() # - получаем позицию мыши
	look_at(mouse_pos) # - персонаж смотрит на курсор (мышь)
	$AnimatedSprite2D.global_rotation = 0.0 # - блокирует возможность вращения текстуры персонажа (если в этом есть необходимость)
```

> [!CAUTION]
> Методы написания скрипта может отличаться в зависимости от текстур, которые вы используете. Некоторые текстуры может быть необходимо вращать, а некоторые - нет. Будьте внимательны и готовы к такому

Так как мы делаем шутер можно добавить удобный прицел. Для этого создадим еще одну переменную

```gdscript
var cursor = preload("res://textures/white_crosshair.png")
```

```gdscript
func _physics_process(delta):
  ...  
  Input.set_custom_mouse_cursor(cursor, Input.CURSOR_ARROW, Vector2(16,16))
```

>[!IMPORTANT]
> Обрати внимание на путь, где лежит текстура. У вас может немного отличаться, но лучше все разбивать на отдельные папки


#### Создание деша
Фрагмент кода ниже нужен для создания деша (ускорения)

![](https://github.com/mykweenn/byteschool/blob/main/shooter/img/tumblr_nemrsjk8hi1sulisxo1_1280.webp)

> У нас будет не так красиво выглядеть деш, но по механике будет так работать

Создадим переменные которые нам понадобятся чуть позже

```gdscript
var dash_trigger = false
var dash_time = 0
```

Теперь создадим функцию рывка
```gdscript
func dash(delta):
	transform.origin = lerp(transform.origin, transform.origin + 40 * input_vector , 0.1)
```

Этот код означает следующее:

* transform.origin - это позиция объекта в пространстве.
* lerp() - это функция, которая выполняет линейную интерполяцию между двумя значениями. В данном случае, она применяется к текущей позиции объекта и новой позиции, которая получается при добавлении вектора управления (input_vector), умноженного на 40 .
* 0.1 - это коэффициент сглаживания, который определяет, насколько быстро объект будет перемещаться к новой позиции.
Таким образом, этот код перемещает объект с текущей позиции к новой позиции, которая получается при добавлении вектора управления, умноженного на 40, с использованием линейной интерполяции для плавного перемещения.

Ниже представлен метод, в котором написаны условия для применения деша

```gdscript
func get_input_velocity(delta): # В аргумент
	if Input.is_action_just_pressed("dash"): # Если нажата кнопка деша, то...
		dash_trigger = true # срабатывает триггер деша
	if dash_trigger == true: # если триггер деша равен истине, то
		dash_time += delta # к дешу прибавляется delta (таким образом создается локальный таймер)
		dash(delta) # вызываем функцию деша, которую вы писали ранее
		if dash_time > 0.25: # Если время деша превышает 0.25 секунды, то
			dash_trigger = false # вырубаем деш
			dash_time = 0 # время деша откатывается к 0
```
Все что нам остается это добавить вызов функции в методе `_physics_process(delta)`

```gdscript
get_input_velocity(delta)
```
И последним что мы сделаем в этом уроке, добавим прицеливание. Для его создания нам понадобится `Marker2D`

![image](https://github.com/Sindikaty/byteschool/assets/158248099/0ab3e2fc-1f94-4943-8e0f-aea472e5e4ac)

Создадим новую функцию `scope`
```gdscript
func scope():
	if Input.is_action_pressed("aim"): # При зажатии ПКМ срабатывает одно из условий ниже
		$Camera2D.position = $scope.position # Если нажат ПКМ, то камера перейдет к позиции прицела
	else:
		$Camera2D.position = Vector2(0,0) # Иначе камера сместится обратно в начальную позицию
```
И добавим ее вызов функции в методе `_physics_process(delta)`

## Урок 2
