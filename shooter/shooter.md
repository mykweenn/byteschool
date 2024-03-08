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
* AudioStreamPlayer2D - Нужен для звука с двухмерным позиционированием
* Sprite - Узел со спрайтом оружия. Из него вызывается пуля
* Marker2D - Нужен для создания позиции прицела, когда игрок нажмет ПКМ (камера смещается на позицию прицела)
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

Давай разберем построчно код ниже. Пояснение написано в комментариях к коду

```gdscript
func _physics_process(delta: float):
	var mouse_pos = get_global_mouse_position() # - получаем позицию мыши
	look_at(mouse_pos) # - персонаж смотрит на курсор (мышь)
	$AnimatedSprite2D.global_rotation = 0.0 # - блокирует возможность вращения текстуры персонажа (если в этом есть необходимость)
```
Код ниже вы уже разбирали на предыдущих уроках

```gdscript
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
Данный фрагмент кода нужен для создания деша (ускорения)



```gdscript
func dash(delta):
	transform.origin = lerp(transform.origin, transform.origin + 40 * input_vector , 0.1)
```
