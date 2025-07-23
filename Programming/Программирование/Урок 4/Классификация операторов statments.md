# Классификация операторов statments на СИ
В языке программирования **C** операторы (**statements**) можно классифицировать следующим образом:  

---

### **1. Выражения (Expression Statements)**  
Любое выражение, завершающееся точкой с запятой `;`.  
- **Примеры:**  
  ```c
  x = 5;          // присваивание
  printf("Hi!");   // вызов функции
  x++;             // инкремент
  ```

---

### **2. Составные операторы (Compound Statements / Blocks)**  
Группа операторов, заключенная в фигурные скобки `{ }`.  
- **Пример:**  
  ```c
  {
      int x = 10;
      printf("%d", x);
  }
  ```

---

### **3. Операторы управления потоком (Control Flow Statements)**  

#### **3.1. Условные операторы**  
- **`if`, `else if`, `else`**  
  ```c
  if (x > 0) {
      printf("Positive");
  } else if (x < 0) {
      printf("Negative");
  } else {
      printf("Zero");
  }
  ```
- **`switch`-`case`**  
  ```c
  switch (value) {
      case 1: printf("One"); break;
      case 2: printf("Two"); break;
      default: printf("Unknown");
  }
  ```

#### **3.2. Циклы**  
- **`while`**  
  ```c
  while (x < 10) {
      printf("%d ", x);
      x++;
  }
  ```
- **`do-while`**  
  ```c
  do {
      printf("%d ", x);
      x++;
  } while (x < 10);
  ```
- **`for`**  
  ```c
  for (int i = 0; i < 10; i++) {
      printf("%d ", i);
  }
  ```

#### **3.3. Операторы перехода**  
- **`break`** – выход из цикла или `switch`  
- **`continue`** – переход к следующей итерации цикла  
- **`goto`** – безусловный переход  
  ```c
  goto label;
  // ...
  label: printf("Jumped here");
  ```

---

### **4. Операторы возврата (Return Statements)**  
Завершают выполнение функции и возвращают значение.  
- **Пример:**  
  ```c
  int sum(int a, int b) {
      return a + b;
  }
  ```

---

### **5. Пустые операторы (Null Statements)**  
Просто точка с запятой `;` (обычно используются в циклах).  
- **Пример:**  
  ```c
  for (; x < 100; x++);  // цикл без тела
  ```

---

### **6. Операторы объявления (Declaration Statements)**  
Объявление переменных.  
- **Пример:**  
  ```c
  int x = 5;
  char c = 'A';
  ```

---

### **7. Операторы обработки исключений (C99+, через `setjmp`/`longjmp`)**  
В **C** нет `try-catch`, но есть механизмы `setjmp/longjmp` для нелокальных переходов.  
- **Пример:**  
  ```c
  #include <setjmp.h>
  jmp_buf env;
  
  if (setjmp(env) == 0) {
      // код, который может "выбросить" ошибку
      longjmp(env, 1);  // аналог throw
  } else {
      // аналог catch
  }
  ```

---

### **Итоговая классификация:**  
1. **Выражения** (`x = 5;`)  
2. **Составные блоки** (`{ ... }`)  
3. **Управление потоком** (`if`, `switch`, `while`, `for`, `break`, `continue`, `goto`)  
4. **Возврат значения** (`return x;`)  
5. **Пустые операторы** (`;`)  
6. **Объявления переменных** (`int x;`)  
7. **Обработка исключений** (`setjmp/longjmp`)  

Эта классификация охватывает все основные виды операторов в **C**.

	  
# # Классификация операторов statments на Python
В **Python** операторы (statements) можно классифицировать следующим образом:

---

## **1. Простые операторы (Simple Statements)**  
Обычно занимают одну строку и выполняют одно действие.  

### **1.1. Операторы присваивания**  
```python
x = 10  
y, z = 5, "hello"  # множественное присваивание
```

### **1.2. Операторы выражения**  
Любое выражение, которое может быть вычислено.  
```python
print("Hello")  
x + y  
```

### **1.3. Оператор `assert` (проверка условия)**  
Вызывает `AssertionError`, если условие ложно.  
```python
assert x > 0, "x must be positive"  
```

### **1.4. Оператор `pass` (пустая операция)**  
Используется там, где синтаксически требуется оператор, но ничего делать не нужно.  
```python
if x > 0:  
    pass  # заглушка
```

### **1.5. Оператор `del` (удаление ссылки)**  
Удаляет переменную или элемент коллекции.  
```python
del x  
del my_list[0]  
```

### **1.6. Оператор `return` (возврат из функции)**  
```python
def sum(a, b):  
    return a + b  
```

### **1.7. Оператор `yield` (генераторы)**  
Используется в функциях-генераторах.  
```python
def count():  
    yield 1  
    yield 2  
```

### **1.8. Оператор `raise` (вызов исключения)**  
```python
if x < 0:  
    raise ValueError("x cannot be negative")  
```

### **1.9. Оператор `import`**  
```python
import math  
from sys import exit  
```

### **1.10. Оператор `global` и `nonlocal`**  
- `global` – объявляет переменную глобальной.  
- `nonlocal` – позволяет изменять переменную из внешней (неглобальной) области видимости.  
```python
x = 10  
def foo():  
    global x  
    x = 20  

def outer():  
    x = 5  
    def inner():  
        nonlocal x  
        x = 10  
```

---

## **2. Составные операторы (Compound Statements)**  
Содержат вложенные блоки кода (обычно с отступами).  

### **2.1. Условные операторы (`if`, `elif`, `else`)**  
```python
if x > 0:  
    print("Positive")  
elif x < 0:  
    print("Negative")  
else:  
    print("Zero")  
```

### **2.2. Циклы (`for`, `while`)**  
```python
for i in range(5):  
    print(i)  

while x > 0:  
    x -= 1  
```

### **2.3. Операторы управления циклом (`break`, `continue`, `else` для циклов)**  
- `break` – выход из цикла.  
- `continue` – переход к следующей итерации.  
- `else` – выполняется, если цикл завершился без `break`.  
```python
for i in range(10):  
    if i == 5:  
        break  
else:  
    print("Loop finished")  
```

### **2.4. Оператор `try`-`except`-`finally` (обработка исключений)**  
```python
try:  
    x = 1 / 0  
except ZeroDivisionError:  
    print("Division by zero!")  
finally:  
    print("Cleanup")  
```

### **2.5. Оператор `with` (контекстные менеджеры)**  
Используется для работы с ресурсами (файлы, соединения и т. д.).  
```python
with open("file.txt", "r") as f:  
    data = f.read()  
```

### **2.6. Оператор `def` (объявление функций)**  
```python
def greet(name):  
    return f"Hello, {name}!"  
```

### **2.7. Оператор `class` (объявление классов)**  
```python
class Person:  
    def __init__(self, name):  
        self.name = name  
```

### **2.8. Оператор `match`-`case` (структурное сопоставление, Python 3.10+)**  
Аналог `switch` из других языков.  
```python
match value:  
    case 1:  
        print("One")  
    case _:  
        print("Unknown")  
```

---

## **3. Пустые операторы**  
- `pass` (уже упоминался)  
- `...` (Ellipsis, иногда используется как заглушка)  
```python
def unfinished():  
    ...  
```

---

## **Итоговая классификация:**  
1. **Простые операторы**  
   - Присваивание (`x = 10`)  
   - Выражения (`print(x)`)  
   - Управление областями видимости (`global`, `nonlocal`)  
   - Импорт (`import math`)  
   - Исключения (`raise`, `assert`)  
   - Управление памятью (`del`)  
   - Заглушки (`pass`, `...`)  

2. **Составные операторы**  
   - Условные (`if`-`elif`-`else`)  
   - Циклы (`for`, `while`)  
   - Обработка исключений (`try`-`except`-`finally`)  
   - Контекстные менеджеры (`with`)  
   - Объявление функций и классов (`def`, `class`)  
   - Сопоставление (`match`-`case`)  

3. **Операторы управления потоком**  
   - `break`, `continue`, `return`, `yield`  

Эта классификация охватывает все основные виды операторов в **Python**.