Простой и самый понятный пример Observer
Например с Продуктами и покупателями
### Этот код является push-observer
```c#
namespace ObserverProduct
{
    interface IObserver
    {
        void Update(string name, double price);
    }

    interface ISubject
    {
        void AddObserver(IObserver o);
        void RemoveObserver(IObserver o);
        void NotifyObserver();
    }

    class Product : ISubject
    {
        private List<IObserver> observers;
        private string _name;
        private double _price;

        public Product(string name, double price) { 
            observers = new List<IObserver>();  
            _name = name;
            _price = price;
        }
        public void AddObserver(IObserver o) 
        {
            observers.Add(o);
        }
        public void RemoveObserver(IObserver o) 
        {
            observers.Remove(o);
        }
        public void NotifyObserver() 
        {
            foreach(IObserver observer in observers)
            {
                observer.Update(_name, _price);
            }
        }

        public double GetPrice() { return _price; }
        public string GetName() { return _name; }
        public void ChangePrice(double price) 
        {
            _price = price;
            NotifyObserver();
        }
    }

    class WholesaleByer : IObserver
    {
        private ISubject _product;
        public WholesaleByer(ISubject obj) 
        {
            _product = obj;
            obj.AddObserver(this);
        }
        public void Update(string name, double price) 
        {
            if (price < 1000) {
                Console.WriteLine($"WholesaleByer: покупает товар {name} за цену {price}");
            }
        }
    }

    class SimpleByer : IObserver
    {
        private ISubject _product;
        public SimpleByer(ISubject obj)
        {
            _product = obj;
            obj.AddObserver(this);
        }
        public void Update(string name, double price)
        {
            if (price < 500)
            {
                Console.WriteLine($"SimpleByer: покупает товар {name} за цену {price}");
            }
        }
    }

    class RichByer : IObserver
    {
        private ISubject _product;
        public RichByer(ISubject obj)
        {
            _product = obj;
            obj.AddObserver(this);
        }
        public void Update(string name, double price)
        {
            if (price > 1000)
            {
                Console.WriteLine($"RichByer: покупает товар {name} за цену {price}");
            }
        }
    }

    internal class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Stors: Hello, World!");

            var phone = new Product("Телефон", 853);

            RichByer richByer = new RichByer(phone);
            SimpleByer simpleByer = new SimpleByer(phone);
            WholesaleByer wholesaleByer = new WholesaleByer(phone);

            phone.ChangePrice(853);
            phone.ChangePrice(1000);
            phone.ChangePrice(300);
            phone.ChangePrice(3700);

        }
    }
}
```

Эта реализация относится к типу Push-Observer, так как субъект (Product) активно уведомляет наблюдателей (WholesaleByer, SimpleByer, RichByer) о каждом изменении цены товара, передавая им новое значение цены в метод Update.


Данная реализация относится к типу Push-Observer, так как субъект (Product) активно уведомляет наблюдателей (WholesaleByer, SimpleByer, RichByer) о каждом изменении цены товара, передавая им новое значение цены в метод Update. При этом наблюдатели не запрашивают у субъекта информацию о цене, а получают ее от него непосредственно. Таким образом, информация "толкается" от субъекта к наблюдателям, что соответствует паттерну Push-Observer.


###  Этот код является pull-observer

```c#

interface IProduct
{
    void Attach(IBuyer buyer);
    void Detach(IBuyer buyer);
    double GetPrice();
}

class Product : IProduct
{
    private List<IBuyer> buyers = new List<IBuyer>();
    private double price;

    public void Attach(IBuyer buyer)
    {
        buyers.Add(buyer);
    }

    public void Detach(IBuyer buyer)
    {
        buyers.Remove(buyer);
    }

    public double GetPrice()
    {
        return price;
    }

    public void SetPrice(double price)
    {
        this.price = price;
        foreach (IBuyer buyer in buyers)
        {
            buyer.Update(this);
        }
    }
}

interface IBuyer
{
    void Update(IProduct product);
}

class WholesaleBuyer : IBuyer
{
    private double maxPrice;

    public WholesaleBuyer(double maxPrice)
    {
        this.maxPrice = maxPrice;
    }

    public void Update(IProduct product)
    {
        double price = product.GetPrice();
        if (price < maxPrice)
        {
            Console.WriteLine($"WholesaleBuyer: покупает товар за цену {price}");
        }
    }
}

class SimpleBuyer : IBuyer
{
    private double maxPrice;

    public SimpleBuyer(double maxPrice)
    {
        this.maxPrice = maxPrice;
    }

    public void Update(IProduct product)
    {
        double price = product.GetPrice();
        if (price < maxPrice)
        {
            Console.WriteLine($"SimpleBuyer: покупает товар за цену {price}");
        }
    }
}

class RichBuyer : IBuyer
{
    private double minPrice;

    public RichBuyer(double minPrice)
    {
        this.minPrice = minPrice;
    }

    public void Update(IProduct product)
    {
        double price = product.GetPrice();
        if (price > minPrice)
        {
            Console.WriteLine($"RichBuyer: покупает товар за цену {price}");
        }
    }
}

internal class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Stores: Hello, World!");

        var phone = new Product();
        phone.Attach(new RichBuyer(1000));
        phone.Attach(new SimpleBuyer(500));
        phone.Attach(new WholesaleBuyer(1000));

        phone.SetPrice(853);
        phone.SetPrice(1000);
        phone.SetPrice(300);
        phone.SetPrice(3700);
    }
}
```

Данная реализация относится к типу Pull-Observer, так как наблюдатели (WholesaleByer, SimpleByer, RichByer) запрашивают у субъекта (Product) информацию о цене товара, вызывая метод GetPrice. При этом субъект не активно уведомляет наблюдателей о каждом изменении цены, а лишь хранит текущее значение цены и предоставляет его по запросу. Таким образом, информация "тянется" от наблюдателей к субъекту, что соответствует паттерну Pull-Observer.

-----

### Паттерны разработанные на основе **observer**
1. Паттерн Listener (слушатель)
	**Паттерн Listener** (слушатель) также является вариантом паттерна "Наблюдатель". Он представляет собой механизм, который позволяет объектам-слушателям получать уведомления об изменениях состояния объекта-источника.
	
	В отличие от паттерна Pub-Sub, где объекты-подписчики подписываются на каналы, объекты-слушатели регистрируются непосредственно в объекте-источнике событий. Таким образом, когда объект-источник генерирует событие, он оповещает всех зарегистрированных в нем слушателей.
	
	Паттерн Listener может быть полезен в случае, когда не требуется гибкость и масштабируемость, которые предоставляет паттерн Pub-Sub. Например, в простых приложениях, где объект-источник имеет небольшое количество слушателей и не предполагается динамическое добавление или удаление слушателей.
	
	Однако, следует учитывать, что паттерн Listener может привести к жесткой связанности между объектом-источником и объектами-слушателями, что усложняет тестирование и поддержку приложения. Поэтому, при проектировании сложных систем, рекомендуется использовать более гибкие и расширяемые паттерны, такие как Pub-Sub.
2. Паттерн Pub-Sub (издатель-подписчик)
3. Паттерн Медиатор (посредник)
4. Паттерн Модель-Вид-Контроллер (MVC)
	`А как MVC связон с наблюдателем ?`
	
	Паттерн Модель-Вид-Контроллер (MVC) включает в себя паттерн Наблюдатель (Listener), который используется для обновления представления (View) при изменении данных в модели (Model). Контроллер (Controller) является посредником между моделью и представлением, управляя взаимодействием между ними. При изменении данных в модели, наблюдатели, которые были зарегистрированы на этих данных, будут уведомлены, и представление будет обновлено соответствующим образом. Таким образом, паттерн Наблюдатель (Listener) является важной частью паттерна Модель-Вид-Контроллер (MVC).
5. Паттерн Реактивного программирования (Rx)