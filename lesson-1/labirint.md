# Лабиринт
Первый проект ребят, направленный на создание движения персонажа, изучение и тренировки работы с узлами.
Т.к. компьютер будет общий, пусть создадут в папке `Student` свои папки и все проекты будут содержать там. <br>
## 1 урок
### Создание уровня и лабиринта
На этом проекте мы еще не работаем с камерой и поэтому игра будет ограничена в пределах игрового экрана, который видно в движке<br>
<br>
<img src="https://github.com/mykweenn/byteschool/blob/main/lesson-1/img/MpGhqF4Z-gY.jpg?raw=true">
>Естественно, надо объяснить, что за прямоугольник они видят.


В качестве корневого узла мы выбираем 2D ноду<br>
<br>
<img src="https://github.com/mykweenn/byteschool/blob/main/lesson-1/img/sX-LqnN2YtY.jpg?raw=true">
> [!IMPORTANT]
> Также немаловажно объяснить им, что такое [нода](https://docs.godotengine.org/ru/4.x/classes/class_node.html)
> Если вкратце, то нода это узел, а узел - объект, соединённый с другими объектами (узлами) как часть системы
```mermaid
graph TD;
    Узел_A-->Узел_B;
    Узел_A-->Узел_C;
    Узел_B-->Узел_D;
    Узел_C-->Узел_E;
```

Стены будем делать из `ColorRect` <br>
`ColorRect` - просто квадрат, который можно закрасить<br>
Строим лабиринт из ColorRect’ов.<br>

<img src="https://github.com/mykweenn/byteschool/blob/main/lesson-1/img/11wgoNB8Pus.jpg">

> [!TIP]
> Лучше всего однотипные узлы сгруппировать в `node_2d` и переименовать его в говорящее название типа `level` или `walls`.<br>
> Типа вот такого:

<details>

<summary>level</summary>

ColorRect<br> 
ColorRect<br> 
ColorRect<br> 
ColorRect<br> 

</details>

Добавляем столько, сколько нужно. Инструменты для изменения размеров `ColorRect` представлены вверху сцены<br>

<img src = "https://github.com/mykweenn/byteschool/blob/main/lesson-1/img/DzQ0JW6d0Bo.jpg">

1. Курсор. Самый основной инструмент
1. Режим перемещения.
1. Поворот вокруг оси
1. Расширение объекта

### Создаем игрока
Создаем основные узлы для персонажа, под контролем игрока.<br>
  📦player `Корневой узел сцены игрока`<br>
    ┣- 📂Sprite `Спрайт`<br>
    ┣- 📂CollisionShape2D`Коллизия тела`<br>
Узел `CharacterBody2D` в Godot используется для создания персонажей, которые могут перемещаться по игровому полю. Он обеспечивает управление движением персонажа, а также обнаружение столкновений с другими объектами.<br>
Узел `CollisionShape2D` используется для создания коллизий объектов в игре. Он позволяет определить форму и размер объекта, что позволяет обнаруживать столкновения с другими объектами и реагировать на них.<br>
Узел `Sprite` содержит в себе картинку.<br>

### Переходим к работе со скриптом
Создаем скрипт у `Node2D`. Про создание скрипта у игрока пока ничего не говорим, а если кто-то спросит, то сказать, что мы сперва напишем управление из корневого узла и после этого можем порассуждать на эту тему (почему стоит писать скрипт для управления все же в игроке, а не в корневом узле уровня).<br>
```gdscript
func _ready():
  print("Hello world!")

func _process():
  print("I'm working all the time!")
```
Пусть попробуют вызвать сообщение в консоль через метод ready, а потом через process.<br>
Будет необходимо разъяснить этот момент: разницу между двумя методами: ready и process.<br>

Метод `ready` и метод `process` являются двумя основными методами в [Godot Engine](https://godotengine.org/), используемыми для определения логики поведения объектов в игре.<br>
Метод `ready` вызывается один раз во время инициализации объекта и используется для настройки начальных параметров и компонентов объекта. В этом методе вы можете инициализировать переменные, создавать дочерние объекты, устанавливать коллайдеры и т.д. Он выполняется до начала игрового цикла и предоставляет возможность подготовить объект к работе.<br>

Метод `process` вызывается на каждом кадре отрисовки и используется для обновления состояния объекта. В этом методе вы можете добавить логику перемещения, анимаций, проверки столкновений и обработки ввода пользователя. Логика, размещенная в методе `process`, будет выполнена на каждом кадре, поэтому он обеспечивает непрерывное обновление состояния объекта.<br>
Разница между `ready` и `process` состоит в том, что `ready` вызывается только один раз при инициализации объекта, а `process` вызывается на каждом кадре отрисовки. `ready` используется для настройки начальных параметров, а `process` - для обновления объекта во время игры.<br>

> [!IMPORTANT]
> Оба метода являются важными для разработки игр в Godot Engine и предоставляют гибкую архитектуру для определения логики объектов и контроля их поведения.<br>

>Задаем логичный вопрос ученикам: Если мы хотим добавить управление игроку, то что нам для этого нужно? Что сделать чтобы персонаж начал двигаться?
Верно. Добавить управление. Поэтому добавим его через `Проект` -> `Настройки проекта` -> `Список действий`. В открывшемся окне находим поле Добавить новое действие , прописываем его имя: right. Затем добавляем кнопку управления.<br>

Переходим в скрипт и можем порассуждать всегда ли игрок нажимает на кнопки? (наводим на мысль, что нам нужно условие нажатия кнопки, ведь игрок может нажать на кнопку, а может не нажать!)
```gdscript
func _process():
  if Input.is_action_just_pressed("right"):
```
Оформляем в условие и разберем компоненты этой строки<br>
Вот компоненты этой строки:<br>

- `if`  : это условный оператор, который позволяет выполнять определенный блок кода только при выполнении определенного условия.<br>

- `Input`: Это встроенный класс в [Godot Engine](https://godotengine.org/), который предоставляет функционал для работы с вводом от пользователя. Класс `Input` содержит набор методов для проверки состояния клавиш, кнопок, мыши и тачскрина.<br>

* Классы в программировании - это шаблоны или модели, которые определяют состояние и поведение объектов. Они представляют собой абстрактные типы данных, которые могут содержать свойства (переменные) и методы (функции), которые могут быть использованы для создания экземпляров объектов. Классы могут быть наследованы другими классами, что позволяет создавать более сложные структуры и иерархии объектов. Они являются одним из основных инструментов объектно-ориентированного программирования.<br>

* Методы и функции это два разных термина, но они могут выполнять схожие задачи. <br>
* Функция - это блок кода, который может принимать аргументы, обрабатывать их и возвращать результат. Функции могут быть вызваны из любой части программы.<br>
* Метод - это функция, которая определена внутри класса и может быть вызвана только у объектов этого класса. Методы могут иметь доступ к свойствам объекта и изменять их состояние.<br>

Таким образом, методы являются частным случаем функций, определенных внутри класса.<br>

- `is_action_pressed`: Это метод класса `Input`, который используется для проверки состояния нажатой кнопки или клавиши. Он принимает строковый аргумент - название "действия", и возвращает `true`, если данное действие (кнопка или клавиша) в данный момент нажата, и `false`, если она не нажата. В данном случае, "**right**" - это имя действия, которое, связано с кнопкой или клавишей, отвечающей за перемещение вправо.<br>

Таким образом, строка `Input.is_action_pressed("right")` выполняет проверку состояния нажатой кнопки или клавиши с именем "right". Если данное действие активно (кнопка или клавиша нажата), то метод вернет `true`, иначе вернет `false`. Это может быть использовано для управления перемещением или выполнения других действий в игре на основе состояния кнопок или клавиш.
> [!Tip]
>  Можно показать сперва другой способ через is_action_pressed и пусть посмотрят разницу.<br>
```gdscript
func _process():
  if Input.is_action_just_pressed("right"):
    $player.position.x += 2
```

Так как избрано движение вправо (или нет, если они решили сделать движение в любую другую сторону), то выбираем соответствующий вектор (в примере это ось Х и будет координата по этой оси увеличиваться => двигаться вправо).
Движение во все другие стороны оставшиеся пусть пробуют сами, если прошло достаточно много времени (от 5 минут), то помогаем.<br>
Можно заметить, что мы часто используем число 2, то тогда его можно поместить в отдельную переменную для нашего удобства. 
> [!CAUTION]
> Названия переменных должны отражать содержание или смысл переменной. Поэтому без всяких var aboba = 2 и т.д.<br>

```gdscript
var speed = 2


func _process():
  if Input.is_action_just_pressed("right"):
      $player.position.x += speed
  if Input.is_action_just_pressed("left"):
      $player.position.x -= speed
  if Input.is_action_just_pressed("down"):
      $player.position.y += speed
  if Input.is_action_just_pressed("up"):
      $player.position.y -= speed
```

Пока стены мы делать не будем, но сделаем финиш. 
>Тут обсуждаем с ними, как это можно сделать. Скорее всего, я почти уверен, они скажут, что можно сделать через какую-то координату (могут записать ее в переменную или просто сказать, а могут просто молчать).<br>
Пускаем их по ложному следу. Предлагаем сделать конкретную точку с координатами <br>

```gdscript
  if $player.position == Vector2(47, 26):
    print("Win!")
```
<br>
<br>

## 2 урок

1. В ареа2д добавляем узел body_entered() и делаем обновление сцены
```gdscript
func _on_area_2d_body_entered(body):
  get_tree().reload_current_scene()
```

2. Делаем менюшку победы. Для этого добавляем еще 1 ареа2д и добавляем ему коллизию + переименовываем. Добавляем label и кнопку. И пишем следующий код. Добавляем узел body_entered() и добавляем кнопке функционал на обновление сцены
```gdscript
func _on_win_body_entered(body):
  $Label.visible = true
  $Button.visible = true
  set_process(false)

func _on_button_pressed():
  get_tree().reload_current_scene()
```

3. Добавляем таймер. Для этого добавляем Label и в параметре Text пишем Time:
В коде создаем переменную типа float
```gdscript
var timer : float = 0.0
```
И пишем следующий код. 
> [!IMPORTANT]
> Дельта это время с прошедшего кадра
```gdscript
timer += delta
$Labe.text = str("Time: ", int(timer))
```
4. Можно дать попробовать самим, если не получится показать как
```gdscript
if timer >= 2:
  $Button.visible = true
```

Ссылка на код:
https://gist.github.com/mykweenn/77992373e8eccb0b494c30e49c4e09f4.js
