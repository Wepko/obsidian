	1. Что такое Паттерн Revealing Constructor - открытый конструктор?
	2. И почему его нету в списке паттернов асинхронного программирование ? 

%%[[паттерны асинхронного программирование]]%%


1. **Паттерн Revealing Constructor** - это паттерн объектно-ориентированного программирования, который заключается в том, чтобы создавать объекты с помощью конструктора, который принимает не все аргументы сразу, а только некоторые из них, а затем предоставляет методы для установки оставшихся значений. Это позволяет контролировать порядок установки значений и обеспечивает более гибкий и безопасный способ создания объектов.

2. Паттерн Revealing Constructor не относится непосредственно к асинхронному программированию, поэтому его нет в списке паттернов асинхронного программирования. Однако, он может использоваться в любых типах программирования, включая асинхронное программирование, если это соответствует потребностям проекта.

-------
### Применение Revealing constructor к Асинхронному программированию и техникам

>Паттерн вида new Promise( function(..){ .. } ) обычно называется раскрывающим конструктором (revealing constructor). Переданная функция выполняется немедленно (без асинхронной задержки, как обратные вызовы then(..)), и ей передаются два параметра, которым в нашем случае были присвоены имена resolve и reject. Это функции разрешения. resolve(..) обычно сигнализирует о выполнении, а reject(..) сигнализирует об отказе.


