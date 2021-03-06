Система типов Си — реализация понятия типа данных в языке программирования Си. Сам язык предоставляет базовые арифметические типы, а также синтаксис для создания массивов и составных типов. Некоторые заголовочные файлы из стандартной библиотеки Си содержат определения типов с дополнительными свойствами[1][2].

￼
Содержание
1	Базовые типы
1.1	Логический тип
1.2	Типы размера и отступа указателя
1.3	Интерфейс к свойствам базовых типов
2	Целые типы фиксированной длины
2.1	Спецификаторы формата для printf и scanf
3	Структуры
4	Массивы
5	Типы указателей
6	Объединения
7	Перечисления
8	Указатели на функции
9	Квалификаторы типов
10	Классы хранения


#### Структуры
Структуры в Си позволяют хранить несколько полей и в одной переменной. В других языках могут называться записями или кортежами. Например, данная структура хранит в себе имя человека и дату рождения:

```
struct birthday
{
    char name[20];
    int day;
    int month;
    int year;
};
```

Объявление структур в теле программы всегда должно начинаться с ключевого struct (необязательно в C++). Доступ к элементам структуры осуществляется с помощью оператора . или ->, если мы работаем с указателем на структуру. Структуры могут содержать указатели на самих себя, что позволяет реализовывать многие структуры данных, основанных на связных списках. Такая возможность может показаться противоречивой, однако все указатели занимают одинаковое число байт, поэтому размер этого поля не изменится от числа полей структуры.

Структуры не всегда занимают число байт, равное сумме байт их элементов. Компилятор обычно выравнивает элементы в блоки по 4 байта. Также есть возможность ограничить число бит, отводимое на конкретное поле, для этого надо после имени поля через двоеточие указать размер поля в битах. Такая возможность позволяет создавать битовые поля.

Некоторые особенности структур:

Массивы[править | править код]
Для каждого типа T, кроме void и типов функций, существует тип «массив из N элементов типа T». Массив — это коллекция значений одного типа, хранящихся последовательно в памяти. Массив размера N индексируется целым числом от 0 до N-1. Также возможны массивы, с неизвестным для компилятора размером. В роли размера массива должна выступать константа. Примеры

int cat[10] = {5,7,2};  // массив из 10 элементов, каждый типа int
int bob[];    // массив с неизвестным количеством элементов типа 'int'.
Массивы могут быть инициализированы с помощью списка инициализации, но не могут быть присвоены друг к другу. Массивы передаются в функции, с помощью указателя на первый элемент (имя массива и есть адрес первого элемента). Многомерные массивы являются массивами массивов. Примеры:

int a[10][8];  // массив из 10 элементов, каждый типа 'массив из 8 int элементов'
float f[][32] = {{0},{4,5,6}};
Типы указателей[править | править код]
Для любого типа T существует тип «указатель на T».

Переменные могут быть объявлены как указатели на значения различных типов с помощью символа *. Для того чтобы определить тип переменной как указатель, нужно предварить её имя звёздочкой.

char letterC = 'C';
char *letter = &letterC; //взятие адреса переменной letterC и присваивание в переменную letter
printf("This code is written in %c.", *letter); //"This code is written in C."
Помимо стандартных типов, можно объявлять указатели на структуры и объединения:

struct Point { int x,y; } A;
A.x = 12;
A.y = 34;
struct Point *p = &A;
printf("X: %d, Y: %d", (*p).x, (*p).y); //"X: 12, Y: 34"
Для обращения к полям структуры по указателю существует оператор «стрелочка» ->, синонимичный предыдущей записи: (*p).x — то же самое, что и p->x.

Поскольку указатель — тоже тип переменной, правило «для любого типа T» выполняется и для них: можно объявлять указатели на указатели. К примеру, можно пользоваться int***:

int w = 100;
int *x = &w;
int **y = &x;
int ***z = &y;
printf("w contains %d.", ***z); //"w contains 100."
Существуют также указатели на массивы и на функции. Указатели на массивы имеют следующий синтаксис:

char *pc[10]; // массив из 10 указателей на char
char (*pa)[10]; // указатель на массив из 10 переменных типа char
pc — массив указателей, занимающий 10 * sizeof(char*) байт (на распространённых платформах — обычно 40 или 80 байт), а pa — это один указатель; занимает он обычно 4 или 8 байт, однако позволяет обращаться к массиву, занимающему 10 байт: sizeof(pa) == sizeof(int*), но sizeof(*pa) == 10 * sizeof(char). Указатели на массивы отличаются от указателей на первый элемент арифметикой. Например, если указатели pa указывает на адрес 2000, то указатель pa+1 будет указывать на адрес 2010.

char (*pa)[10];
char array[10] = "Wikipedia";
pa = &array;
printf("An example for %s.\n", *pa); //"An example for Wikipedia."
printf("%c %c %c", (*pa)[1], (*pa)[3], (*pa)[7]); //"i i i"
Объединения[править | править код]
Объединения — это специальные структуры, которые позволяют различным полям разделять общую память. Таким образом в объединении может храниться только одно из полей. Размер объединения равен размеру наибольшего поля. Пример:

union
{
    int i;
    float f;
    struct
    {
        unsigned int u;
        double d;
    } s;
} u;
В примере выше u по размеру равна u.s (размер которой является суммой u.s.u и u.s.d), так как s больше i и f. Чтение из объединения не включает преобразования типов.

Перечисления[править | править код]
Перечисления позволяют определять в коде пользовательские типы. Пример:

enum
{
    red,
    green=3,
    blue
} color;
Перечисления улучшают читабельность кода, однако они не типобезопасны (например, для системы 3 и green одно и то же. В C++ для исправления этого недостатка были введены enum class), так как являются целочисленными. В данном примере значение red равно нулю, а значение blue четырём.

Указатели на функции[править | править код]
Указатели на функции позволяют передавать одни функции в другие и реализуют механизм обратного вызова. Указатели на функции позволяют ссылаться на функции с определённой сигнатурой. Пример создания указателя на функцию abs, принимающую int и возвращающую int с именем my_int_f:

int (*my_int_f)(int) = &abs;
// оператор & необязателен, но вносит ясность, явно показывая что мы передаём адрес
Указатели на функции вызываются по имени, как обычные вызовы функций. Указатели на функции отделены от обычных указателей и указателей на void.

Более сложный пример:

char ret_a(int x)
{
    return 'a'+x;
}

typedef char (*fptr)(int);

fptr another_func(float a)
{
    return &ret_a;
}
Здесь для удобства мы создали псевдоним с именем fptr для указателя на функцию, возвращающую char и принимающую int. Без typedef синтаксис был бы сложнее для восприятия:

char ret_a(int x)
{
    return 'a'+x;
}

char (*func(float a, int b))(int)
{
    char (*fp)(int) = &ret_a;
    return fp;
}

char (*(*superfunc(double a))(float, int))(int)
{
    char (*(*fpp)(float, int))(int)=&func;
    return fpp;
}
Функция func возвращает не char, как может показаться, а указатель на ф-цию, возвращающую char и принимающую int. И принимает float и int.