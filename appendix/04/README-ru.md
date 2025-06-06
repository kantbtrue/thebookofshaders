## Введение для JavaScript-программистов
автор [Николя Баррадо](http://www.barradeau.com/)


Если вы JavaScript-разработчик, велика вероятность, что вы будете немного озадаченЫ, читая эту книгу. В самом деле, есть множество различий между манипулированием высокоуровневыми абстракциями на JS и ковырянием в шейдерах. Но, в отличие от лежащего на более низком уровне языка ассемблера, GLSL является человекочитаемым, и я уверен, что разобравшись с его особенностями, вы быстро сможете начать его использовать.

Я предполагаю, что у вас есть хотя бы поверхностные знания JavaScript и Canvas API. Если это не так - ничего страшного. Вам всё равно будет интересно читать большую часть этой главы.

Так же, я не буду сильно углубляться в детали, и некоторые вещи могут быть лишь _полуправдой_. Эта глава не является подробным руководством.

JavaScript очень хорош для быстрого прототипирования. Вы можете беспорядочно набросать кучу нетипизированных переменных и методов, динамически добавлять и удалять члены класса, обновить страницу и увидеть как она работает. Затем сделать изменения в соответствии с увиденным, обновить страницу, повторить. Жизнь - простая штука. Так в чём же разница между JavaScript и GLSL? Они оба работают в браузере, оба используются для рисования всяких прикольных штук на экране, и к тому же, JS проще в использовании.

ОСновная разница в том, что JavaScript - **интерпретируемый** язык, в то время как GLSL - **компилируемый**. **Скомпилированная** программа исполняется нативно, она является низкоуровневой и в целом высокопроизводительна. **Интерпретируемая** программа требует [виртуальную машину](https://ru.wikipedia.org/wiki/%D0%92%D0%B8%D1%80%D1%82%D1%83%D0%B0%D0%BB%D1%8C%D0%BD%D0%B0%D1%8F_%D0%BC%D0%B0%D1%88%D0%B8%D0%BD%D0%B0) для своего исполнения, является высокоуровневой и в общем случае более медленной.

Когда браузер (_**виртуальная машина** JavaScript_) **исполняет** или **интерпретирует** кусок кода на JS, он не имеет ни малейшего понятия чем является каждая переменная и что делает каждая функция (за исключением **типизированных массивов**). Поэтому он не может оптимизировать что-либо _наперёд_. Чтение кода браузером занимает какое-то время, чтобы **вывести** (исходя из использования) типы переменных и методов, и, по возможности, преобразовать _часть_ кода в ассемблер, который будет исполняться намного быстрее.

Это медленный, болезненный и до сумасшествия сложный процесс. Если вам интересны подробности, рекомендую посмотреть как [работает движок V8 в Хроме](https://developers.google.com/v8/). Хуже всего то, что браузер оптимизирует JS как ему хочется, и этот процесс _скрыт_ от программиста. Вы бессильны.

**Компилируемая** программа не интерпретируется на ходу. Её запускает операционная система, и она исполняется, если она корректна. Это многое меняет. Если вы забудете точку с запятой в конце строки, ваш код станет некорректным и просто не скомпилируется. Он вообще не превратится в программу.

Это сурово, но это то, чем является **шейдер**: _компилируемая программа для исполнения на GPU_. Не пугайтесь! **Компилятор**, то есть та программа которая проверяет ваш код на корректность, станет вашим лучшим другом. Примеры и [редактор](http://editor.thebookofshaders.com/) в этой книге очень дружественны к пользователю. Она подскажут в каком месте программа не скомпилировалась, и когда после всех правок шейдер будет готов к компиляции, результат его работы будет немедленно отображён. Это отличный способ обучения в силу его наглядности и невозможности что-либо сломать.

И последнее: **шейдер** состоит из двух программ: **вершинного** и **фрагментного** шейдера. Вкратце, вершинный шейдер (первая программа) принимает на вход и преобразовывет *геометрию*, которая затем превращается в последовательность **пикселей** (или **фрагментов**), поступающих на вход второго шейдера. И уже второй шейдер решает в какой цвет нужно покрасить пиксели. Эта книга посвящена именно вторым шейдерам. Во всех примерах геометрия - это прямоугольник, покрывающий всю доступную область.

Готовы?

Поехали!

### Сильная типизация
![первая картинка в Гугле по запросу «сильные типы» на 20 мая 2016](strong_type.jpg)

Когда вы приходите с JS или любого другого нетипизированного языка, **типизирование** переменных является для вас чужеродной концепцией, и это станет сложнейшим шагом при переходе к GLSL. **Типизация**, как легко догадаться, означает, что вам придётся давать **тип** каждой переменной и функции. Отсюда следует, что ключевого слова **`var`** больше не существует. Считайте, что полиция мыслей от GLSL стёрла его из общеупотребимого языка и вы больше не можете его произносить потому что, ну... его не существует.

Вместо использования волшебного слова **`var`** вам придётся _явно указывать тип каждой переменной_, тогда компилятор увидит те объекты и примитивы, с которыми он умеет эффективно обращаться. Обратная сторона невозможности использования ключевого слова **`var`** заключается в том, что вам нужно очень хорошо знать особенности типов всех переменных. Но поверьте, типов в GLSL немного, и они все достаточно просты (GLSL - не Java-фреймворк).

Всё это может выглядеть пугающе, но всё же это не сильно отличается от того, что вы обычно делаете на JS. Например, если переменная булева, то в ней может храниться только `true` или `false`. Если переменная называется `var uid = XXX;`, то в ней вероятно хранится целочисленное значение. Если же она объявлена как `var y = YYY;`, то это _возможно_ ссылка на значение с плавающей точкой. Что ещё лучше, при использовании сильных типов вам не придётся гадать что означает `X == Y`, и означает ли это `typeof X == typeof Y`, или `typeof X !== null && Y...`. В любом случае, вы *знаете* что здесь написано, а если и не знаете, то компилятор знает точно.

Перечислим **скалярные типы** языка GLSL (**скаляр** описывает количество): `bool` (булев тип), `int` (целочисленный) и `float` (значения с плавающей точкой). Есть и другие типы, но пока давайте рассмотрим как объявляются переменные в GLSL:

```glsl
// булево значение
JS: var b = true;               GLSL: bool b = true;

// целое значение
JS: var i = 1;                  GLSL: int i = 1;

// число с плавающей точкой
JS: var f = 3.14159;            GLSL: float f = 3.14159;
```
Не очень трудно, правда? Как было замечено выше, такой подход делает программирование проще, так как вы не тратите время на выслеживание типа какой-либо переменной. Всё ещё сомневаетесь? Помните, что это так же делается для того, чтобы ваша программа исполнялась в разы быстрее, чем на JS.

#### void
В GLSL есть тип `void`, который приблизительно соответствует `null`. Он используется в качестве возвращаемого типа для метода, который не возвращает ничего, и вы не можете объявить переменную этого типа.

#### boolean
Как вам известно, булевы значения в основном используются для проверки условий: `if( myBoolean == true ){}else{}`. Условное ветвление очень легко использовать на CPU, но [параллельная природа](http://thebookofshaders/01/?lan=ru) GLSL делает это утверждение не совсем верным. Как правило, использование условных переходов не рекомендуется, и в книге описано несколько способов обойти это ограничение.

#### приведение типов
Как говорил [Боромир](https://ru.wikipedia.org/wiki/%D0%91%D0%BE%D1%80%D0%BE%D0%BC%D0%B8%D1%80), нельзя просто так взять и смешать типизированные примитивы. В отличие от JavaScript, GLSL не позволит вам выполнять операции между переменными различных типов.

Например вот это:
```glsl
int     i = 2;
float   f = 3.14159;

// попытка умножить целое на значение с плавающей точкой
float   r = i * f;
```
не будет работать, потому что вы пытаетесь скрестить **_кошку_** с **_жирафом_**. Проблема решается с помощью **приведения типов**, которое _заставит компилятор поверить_, что *`i`* имеет тип `float`, не меняя фактический тип *`i`*.
```glsl
//приведение типа целочисленной переменной 'i' к float
float   r = float( i ) * f;
```

Это в точности как переодевание **_кошки_** в **шкуру _жирафа_**, которое будет работать как и ожидается: в `r` сохранится результат умножения `i` x `f`.

Любой из упомянутых выше типов можно **привести** к любому другому. При этом приведение `float` к `int` будет работать как `Math.floor()`, удаляя числа справа от запятой. Приведение `float` или `int` к булеву типу вернёт `true` если переменная не равна нулю.

#### конструктор
**Типы** переменных так же являются **конструкторами классов** для самих себя. Фактически, переменную типа `float` можно представлять как _`экземпляр`_ класса _`float`_.

Следующие объявления равнозначны:

```glsl
int     i = 1;
int     i = int( 1 );
int     i = int( 1.9995 );
int     i = int( true );
```
Для `скалярных` типов это выглядит весьма тривиально, не особо отличаясь от **приведения**, но в этом появится больше смысла когда мы дойдём до раздела о *перегрузках*.

Итак, мы изучили три `примитивных типа`, без которых невозможно обойтись, но в GLSL есть и другие.

### Векторы
![первый результат в Гугле по запросу 'vector villain' на 20 мая 2016](vector.jpg)

Как и JavaScript, в GLSL вам понадобятся более продвинутые способы для манипуляции данными, и здесь **`векторы`** будт очень кстати. Я предполагаю, что вам доводилось писать на JS класс `Point`, который содержит значения `x` и `y`, и выглядит как-то так:
```glsl
// определение:
var Point = function( x, y ){
    this.x = x || 0;
    this.y = y || 0;
}

// объявление экземпляра:
var p = new Point( 100,100 );
```

Как мы только что видели, этот код жутко неправилен на всех уровнях. Во-первых, это ключевое слово **`var`**, затем это ужасающее **`this`** и **нетипизированные** значения `x` и `y`... Нет, такое явно не будет работать в мире шейдеров.

Вместо этого GLSL предоставляет встроенные структуры для группировки данных:

 * `bvec2`: 2D булев вектор, `bvec3`: 3D булев вектор, `bvec4`: 4D булев вектор
 * `ivec2`: 2D целочисленный вектор, `ivec3`: 3D целочисленный вектор, `ivec4`: 4D целочисленный вектор
 * `vec2`: 2D вектор с плавающей точкой, `vec3`: 3Dвектор с плавающей точкой, `vec4`: 4D вектор с плавающей точкой

Вдумчивый читатель заметит, что каждому примитивному типу соответствует **векторный** тип. Из написанного выше легко вывести, что `bvec2` содержит два булевых значения, а `vec4` будет содержать четыре значения в плавающей точкой.

Так же векторы вводят такую величину, как размерность. Это не означает, что вы должны использовать 2D-вектор при отрисовке 2D-графики и 3D при рисовании 3D-изображений. Для чего в таком случае используется четырёхмерный вектор? (ну, на самом деле это называется «тессеракт» или «гиперкуб»)

Нет, **размерность** указывает на количество **компонентов** или **переменных**, хранимых в **векторе**:
```glsl
// объявляем двумерный булев вектор
bvec2 b2 = bvec2 ( true, false );

// объявляем трёхмерный целочисленный вектор
ivec3 i3 = ivec3( 0,0,1 );

// объявляем четырёхмерный вектор значений с плавающей запятой
vec4 v4 = vec4( 0.0, 1.0, 2.0, 1. );
```
`b2` содержит два различных булевых значения, `i3` содержит 3 различных целых, а `v4` содержит 4 различных значения с плавающей точкой.

Но как обратиться к этим значениям?
В случае скаляров ответ очевиден: при объявлении `float f = 1.2;` переменная `f` содержит значение `1.2`. Для **векторов** всё немного по-другому и выглядит это довольно красиво.

#### доступ к элементам векторов
Есть несколько способов доступа к значениям
```glsl
// объявим четырёхмерный вектор значений с плавающей точкой
vec4 v4 = vec4( 0.0, 1.0, 2.0, 3.0 );
```
четыре его значения можно извлечь следующим образом
```glsl
float x = v4.x;     // x = 0.0
float y = v4.y;     // y = 1.0
float z = v4.z;     // z = 2.0
float w = v4.w;     // w = 3.0
```
легко и просто. Ниже приведены равнозначные способы доступа к данным:
```glsl
float x =   v4.x    =   v4.r    =   v4.s    =   v4[0];     // x = 0.0
float y =   v4.y    =   v4.g    =   v4.t    =   v4[1];     // y = 1.0
float z =   v4.z    =   v4.b    =   v4.p    =   v4[2];     // z = 2.0
float w =   v4.w    =   v4.a    =   v4.q    =   v4[3];     // w = 3.0
```

Вдумчивый читатель заметил три факта:
   * `X`, `Y`, `Z` и `W` как правило используются в программах для представления векторов в пространстве
   * `R`, `G`, `B` и `A` используются для кодирования цвета и альфа-канала
   * `[0]`, `[1]`, `[2]` и `[3]` означают, что векторы являются массивами с произвольным доступом

В зависимости от того, работаете ли вы с двух- или трёхмерными координатами, цветом с альфа-каналом или без такового, или просто какими-то произвольными значениями, вы можете выбрать наиболее подходящий тип и размерность вектора. Обычно координаты и векторы (в геометрическом смысле слова) хранятся как `vec2`, `vec3` или `vec4`, цвета как `vec3` или `vec4`, но в целом никаких ограничений на использование переменных нет. Например, никто не запрещает вам хранить единственное булево значение как `bvec4`, но это приведёт в излишнему расходу памяти.

**Заметим**, что в шейдерах значения цвета (`R`, `G`, `B`, `A`) нормализованы, то есть лежат в диапазоне от 0 до 1, а не от 0 до 0xFF, поэтому для них лучше использовать вещественный тип `vec4`, а не целочисленный `ivec4`.

Уже лучше, но мы идём далее!

#### перемешивание

Из вектора можно извлечь несколько значений одновременно. Например, если вам нужны только `X` и `Y` из `vec4`, на JavaScript вы бы написали что-то вроде этого:
```glsl
var needles = [0, 1]; // размещение 'x' и 'y' в структуре данных
var a = [ 0,1,2,3 ]; // структура данных 'vec4'
var b = a.filter( function( val, i, array ) {
return needles.indexOf( array.indexOf( val ) ) != -1;
});
// b = [ 0, 1 ]

// или более буквально:
var needles = [0, 1];
var a = [ 0,1,2,3 ]; // структура 'vec4'
var b = [ a[ needles[ 0 ] ], a[ needles[ 1 ] ] ]; // b = [ 0, 1 ]
```
Выглядит уродливо. В GLSL данные можно извлечь вот так:
```glsl
// создаём четырёхмерный вектор с плавающей запятой
vec4 v4 = vec4( 0.0, 1.0, 2.0, 3.0 );

// и извлекаем только X и Y
vec2 xy =   v4.xy; //   xy = vec2( 0.0, 1.0 );
```
Что это было?! Когда вы составляете воедино методы доступа к полям, GLSL изящно возвращает запрошенное подмножество в виде значения наиболее подходящего типа. Это возможно, потому что вектор является структурой данных с произвольным доступом, прямо как массив в javaScript. Поэтому, можно не только обратиться к подмножеству данных вектора, но и указать **порядок**, в котором нужно обращаться. Следующий код обратит порядок компонентов вектора:
```glsl
// создаём четырёхкомпонентный вектор R,G,B,A
vec4 color = vec4( 0.2, 0.8, 0.0, 1.0 );

// и извлекаем компоненты цвета в порядке A,B,G,R
vec4 backwards = v4.abgr; // backwards = vec4( 1.0, 0.0, 0.8, 0.2 );
```
И конечно же, к одной компоненте можно обратиться многократно:
```glsl
// создаём четырёхкомпонентный вектор R,G,B,A
vec4 color = vec4( 0.2, 0.8, 0.0, 1.0 );

// и извлекаем vec3 с компонентами GAG на основе каналов G и A исходного цвета
vec3 GAG = v4.gag; // GAG = vec4( 0.8, 1.0, 0.8 );
```

Очень удобно составлять части вектора воедино, извлекать только rgb-компоненты из вектора цвета с прозрачностью и т.п.

#### перегрузим всё!
В разделе о типах я упоминал упоминал **конструкторы** и ещё одно великолепное свойство языка GLSL - **перегрузку**. **Перегрузка** оператора или функции означает _изменение поведения этого оператора или функции в зависимости от операндов/аргументов_. В JavsScript нет перегрузки, поэтому вначале она может показаться вам странной, но немного попользовавшись ей, вы зададитесь вопросом, почему же она не реализована в JavaScript (краткий ответ - *типизация*).

Рассмотрим простейший пример перегрузки:

```glsl
vec2 a = vec2( 1.0, 1.0 );
vec2 b = vec2( 1.0, 1.0 );
// перегруженное сложение
vec2 c = a + b;     // c = vec2( 2.0, 2.0 );
```
ШТОА? Можно складывать сущности, не являющиеся числами?!

Именно. И конечно же, это применимо ко всем операторам (`+`, `-`, `*` и `/`), и это только начало.
Рассмотрим фрагмент кода:
```glsl
vec2 a = vec2( 0.0, 0.0 );
vec2 b = vec2( 1.0, 1.0 );
// перегруженный конструктор
vec4 c = vec4( a , b );         // c = vec4( 0.0, 0.0, 1.0, 1.0 );
```
Мы соорудили `vec4` из двух `vec2`, используя `a.x` и `a.y` в качестве компонент `X` и `Y` для нового вектора `c`. Затем мы взяли `b.x` и `b.y` в качестве `Z` и `W` для `c`.

Так работает перегрузка функции по набору параметров, в данном случае это **конструктор** `vec4`. Это означает, что несколько **версий** одного и того же метода с различными наборами параметров могут мирно сосуществовать в одной программе. Например, все следующие объявления корректны:
```glsl
vec4 a = vec4(1.0, 1.0, 1.0, 1.0);
vec4 a = vec4(1.0);// x, y, z, w all equal 1.0
vec4 a = vec4( v2, float, v4 );// vec4( v2.x, v2.y, float, v4.x );
vec4 a = vec4( v3, float );// vec4( v3.x, v3.y, v3.z, float );
etc.
```
От вас требуется только подать достаточное количество параметров для заполнения **вектора**.

Наконец, вы можете перегружать встроенные функции для тех типов аргументов, для которых они не были изначально задуманы (но лучше не делать этого слишком часто).

#### нужно больше типов
Векторы прикольные. Они - мышцы вашего шейдера. Но есть и другие типы, например матрицы и текстурные семплеры, о которых будет рассказано ниже.

В GLSL есть массивы. Конечно же, они типизированные, и у них есть несколько отличий от массивов в JS:
 * у них фиксированный размер
 * вы не можете использовать push(), pop(), splice() и т.п., свойство ```length``` тоже отсутствует
 * их нельзя инициализировать значениями при объявлении
 * значения нужно задавать по одному

вот это работать не будет:
```glsl
int values[3] = [0,0,0];
```
а вот это заработает:
```glsl
int values[3];
values[0] = 0;
values[1] = 0;
values[2] = 0;
```
Этого хватает, если вы знаете все ваши данные или работаете с небольшими массивами данных. Если вам нужно больше выразительности, вы можете использовать структуры (```struct```). Они похожи на _объекты_ без методов. Они позволяют хранить несколько переменных в одном объекте:
```glsl
struct ColorStruct {
    vec3 color0;
    vec3 color1;
    vec3 color2;
}
```
например, вы можете задавать и извлекать значения _цвета_ следующим образом:
```glsl
// инициализируем структуру
ColorStruct sandy = ColorStruct( 	vec3(0.92,0.83,0.60),
                                    vec3(1.,0.94,0.69),
                                    vec3(0.95,0.86,0.69) );

// получем доступ к значениям
sandy.color0 // vec3(0.92,0.83,0.60)
```
Это синтаксический сахар, но он может помочь вам писать более чистый, или как минимум более привычный код.

#### выражения и условия

Структуры данных очень полезны, но рано или поздно  нам _возможно_ понадобится проходить по массиву или выполнять проверку условия. К счастью, синтаксис для этого очень близок к JavaScript.
Условие выглядит так:
```glsl
if( condition ){
    //true
}else{
    //false
}
```
Цикл `for` выглядит так:
```glsl
const int count = 10;
for( int i = 0; i <= count; i++){
    //do something
}
```
пример переменной цикла с плавающей точкой:
```glsl
const float count = 10.;
for( float i = 0.0; i <= count; i+= 1.0 ){
    //do something
}
```
Заметим, что ```count``` должна быть объявлена константой. Это означает, что перед её объявлением должен быть **квалификатор** ```const```, который будет рассмотрен чуть ниже.

Так же нам доступны ключевые слова ```break``` и ```continue```:
```glsl
const float count = 10.;
for( float i = 0.0; i <= count; i+= 1.0 ){
    if( i < 5. )continue;
    if( i >= 8. )break;
}
```
Имейте ввиду, что на некоторых типах оборудования ```break``` не работает ожидаемым образом и не прерывает цикл заранее.

В целом, старайтесь делать количество итераций как можно меньше, и избегайте циклов и ветвлений как можно чаще.

#### квалификаторы

Помимо типов переменных в GLSL есть **квалификаторы**. Вкратце, квалификаторы сообщают компилятору какая переменная для чего предназначена. Например, некоторые данные для GPU могут приходить только со стороны CPU. Такие данные называются **атрибутами** и **юниформами**. **Атрибуты** встречаются только в вершинных шейдерах, а **юниформы** - и в вершинных, и во фрагментных. Так же есть квалификатор ```varying```, используемый для передачи переменных от вершинного шейдера к фрагментному.

Я не буду сильно углубляться в подробности, ибо мы в основном рассматриваем **фрагментные шейдеры*, но далее в книге вам возможно встретится что-то вроде
```glsl
uniform vec2 u_resolution;
```
Что здесь происходит? Мы задали квалификатор ```uniform``` перед типом переменной, указав, что разрешение изображения передаётся в шейдер из CPU. Ширина изображения находится в `x`-компоненте 2D-вектора, а высота - в `y`-компоненте.

Когда компилятор видит переменную, объявленную с этим квалификатором, он сделает чтобы вы не могли *записать* это значение в рантайме.

То же самое применимо к переменной ```count```, которая была пороговым значением в цикле ```for```:
```glsl
const float count = 10.;
for( ... )
```
Когда мы используем квалификатор ```const```, компилятор не даёт нам перезаписать значение, которое в противном случае не было бы константой.

Ещё три квалификатора используются в сигнатурах функций: ```in```, ```out``` и ```inout```. В JavaScript переданные в функцию аргументы предназначены только для чтения. Их изменение внутри функции не приводит к изменению значений за её пределами.
```glsl
function banana( a ){
    a += 1;
}
var value = 0;
banana( value );
console.log( value );// > 0 ; значение за пределами функции не изменилось
```

Используя квалификаторы аргументов, можно изменять их поведение:
  * ```in``` предназначен только для чтения (по умолчанию)
  * ```out```  только для записи: значение такого аргумента нельзя прочитать, но можно записать
  * ```inout```  чтение и запись

Перепишем упомянутый выше метод на GLSL:
```glsl
void banana( inout float a ){
    a += 1.;
}
float A = 0.;
banana( A ); // теперь A = 1.;
```
Это поведение сильно отличается от JS и даёт множество возможностей. При этом, не обязательно всегда указывать квалификаторы аргументов. По умолчанию аргументы предназначены только для чтения.

#### пространство и координаты

Напоследок заметим, что в DOM и Canvas 2D ось Y направлена вниз. Это имеет смысл в контексте DOM, ибо соответствует тому, как свёрстана web-страница: навигационная панель наверху, а контент прокручивается вниз. В webgl-элементе ось Y перевёрнута и указывает вверх.

Это означает, что начало координат (точка (0,0)) расположено в левом нижнем, а не в левом верхнем углу контекста. Текстурные координаты так же следуют этому правилу, которое на первый взгляд кажется контринтуитивным.

## На этом всё!
Конечно, мы могли мы углубиться во всяческие детали, но, как было сказано вначале, эта статья писалась как простое введение для новичков. Здесь уже написано достаточно, чтобы переваривать это некоторое время, но с терпением и практикой этот язык будет становиться всё более естественным для вас.

Надеюсь, этот текст был полезен, а потому самое время приступить к основному содержимому книги!
