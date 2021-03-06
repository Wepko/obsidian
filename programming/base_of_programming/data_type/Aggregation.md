# Агрегирование (программирование)

В объектно-ориентированном программировании под **агрегированием** (или как его еще называют - делегированием) подразумевают методику создания нового класса из уже существующих классов путём их включения. Об агрегировании также часто говорят как об «отношении принадлежности» по принципу «у машины есть корпус, колёса и двигатель».

Вложенные объекты нового класса обычно объявляются закрытыми, что делает их недоступными для прикладных программистов, работающих с классом. С другой стороны, создатель класса может изменять эти объекты, не нарушая при этом работы существующего клиентского кода. Кроме того, замена вложенных объектов на стадии выполнения программы позволяет динамически изменять её поведение. Механизм наследования такой гибкостью не обладает, поскольку для производных классов устанавливаются ограничения, проверяемые на стадии компиляции.

На базе агрегирования реализуется методика делегирования, когда поставленная перед внешним объектом задача перепоручается внутреннему объекту, специализирующемуся на решении задач такого рода.


Агрегация: профессора - факультеты, профессора остаются жить после разрушения факультета
Композиция: университет - факультеты, факультеты без университета погибают. 

![Aggregation-Composition3.png](https://upload.wikimedia.org/wikipedia/commons/d/d0/Aggregation-Composition3.png)

Агрегация (агрегирование по ссылке) — отношение «часть-целое» между двумя равноправными объектами, когда один объект (контейнер) имеет ссылку на другой объект. Оба объекта могут существовать независимо: если контейнер будет уничтожен, то его содержимое — нет.

```
class Professor;
 
class Department
{
  private:
    Professor* members[5];  // Aggregation, т.к. нет оператора delete
};
class Ehe // Пример агрегации
{
private:
    Person& _partner1; // Enthaltener Teil.  // Aggregation
    Person& _partner2; // Enthaltener Teil.  // Aggregation
 
public:
    // Конструктор
    Ehe (Person& partner1, Person& partner2)
        : _partner1(partner1), _partner2(partner2)
    { }
};
```

## **Композиция**
**Композиция** (агрегирование по значению) — более строгий вариант агрегирования, когда включаемый объект может существовать только как часть контейнера. Если контейнер будет уничтожен, то и включённый объект тоже будет уничтожен.

```
class Department;
 
class University
{
  private:
    Department faculty[20];  // Composition
};
class Carburetor;

class Automobile
{
  private:
    Carburetor* itsCarb;  
  public:   
    Automobile() {itsCarb=new Carburetor();}
    virtual ~Automobile() {delete itsCarb;} // Composition, т.к. объект  itsCarb  будет удалён 
};

```