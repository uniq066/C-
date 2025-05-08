# C#

## Создание перечисляемого типа данных (поддержка `foreach`) в C#

Чтобы класс поддерживал цикл `foreach`, он должен реализовывать интерфейс `IEnumerable` (или `IEnumerable<T>`).  Этот интерфейс требует наличия метода `GetEnumerator()`, который возвращает объект, реализующий интерфейс `IEnumerator` (или `IEnumerator<T>`).  `IEnumerator` предоставляет методы для навигации по коллекции.

```csharp
using System;
using System.Collections;
using System.Collections.Generic;

public class MyCollection<T> : IEnumerable<T>
{
    private T[] data;
    private int size;

    public MyCollection(int capacity)
    {
        data = new T[capacity];
        size = 0;
    }

    public void Add(T item)
    {
        if (size == data.Length)
        {
            // При необходимости расширяем массив (можно использовать Array.Resize())
            Array.Resize(ref data, data.Length * 2);
        }
        data[size] = item;
        size++;
    }

    // Реализация интерфейса IEnumerable<T>
    public IEnumerator<T> GetEnumerator()
    {
        return new MyCollectionEnumerator<T>(data, size);
    }

    // Явная реализация интерфейса IEnumerable (не generic)
    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }

    // Вложенный класс для реализации IEnumerator
    private class MyCollectionEnumerator<T> : IEnumerator<T>
    {
        private T[] _data;
        private int _size;
        private int _currentIndex;

        public MyCollectionEnumerator(T[] data, int size)
        {
            _data = data;
            _size = size;
            _currentIndex = -1; // Начинаем с -1, чтобы MoveNext переместился на первый элемент
        }

        public T Current
        {
            get
            {
                if (_currentIndex < 0 || _currentIndex >= _size)
                {
                    throw new InvalidOperationException("Enumerator is either before the first element or after the last element.");
                }
                return _data[_currentIndex];
            }
        }

        object IEnumerator.Current => Current;

        public void Dispose() { } // Ничего не делаем, так как нет неуправляемых ресурсов

        public bool MoveNext()
        {
            _currentIndex++;
            return _currentIndex < _size;
        }

        public void Reset()
        {
            _currentIndex = -1;
        }
    }
}

// Пример использования
public class Example
{
    public static void Main(string[] args)
    {
        MyCollection<string> names = new MyCollection<string>(5);
        names.Add("Alice");
        names.Add("Bob");
        names.Add("Charlie");

        foreach (string name in names)
        {
            Console.WriteLine(name);
        }
    }
}
```

**Пояснения:**

1.  **`MyCollection<T>`:**  Это наш класс, который будет поддерживать `foreach`.  Он хранит данные в массиве `data`.
2.  **`IEnumerable<T>`:**  Мы реализуем этот интерфейс, чтобы указать, что наш класс может быть перечислен.
3.  **`GetEnumerator()`:** Этот метод возвращает объект, который реализует `IEnumerator<T>`. Этот объект отвечает за итерацию по коллекции.
4.  **`MyCollectionEnumerator<T>`:** Это вложенный класс, который реализует `IEnumerator<T>`. Он содержит логику перемещения по коллекции и возврата текущего элемента.
    *   **`Current`:**  Возвращает текущий элемент.  Важно проверять, что индекс находится в допустимых пределах.
    *   **`MoveNext()`:**  Перемещает итератор к следующему элементу и возвращает `true`, если есть следующий элемент, и `false`, если достигнут конец коллекции.
    *   **`Reset()`:**  Сбрасывает итератор в начальное состояние (обычно перед первым элементом).
    *   **`Dispose()`:**  Освобождает ресурсы, используемые итератором (в данном случае не требуется).
5.  **`IEnumerable IEnumerable.GetEnumerator()`:**  Явная реализация `IEnumerable` (без generic). Она просто вызывает `GetEnumerator()`.  Это необходимо для совместимости со старым кодом.
6.  **Пример использования:**  В `Main()` мы создаем экземпляр `MyCollection<string>`, добавляем несколько элементов и затем перебираем их с помощью `foreach`.

## Реализация стека на C#

```csharp
using System;

public class MyStack<T>
{
    private T[] _items;
    private int _size; // Количество элементов в стеке
    private const int DefaultCapacity = 4; // Начальная вместимость

    public MyStack() : this(DefaultCapacity) { }

    public MyStack(int capacity)
    {
        if (capacity < 0)
        {
            throw new ArgumentOutOfRangeException("capacity", "Capacity must be non-negative.");
        }
        _items = new T[capacity];
        _size = 0;
    }

    public int Count
    {
        get { return _size; }
    }

    public void Push(T item)
    {
        if (_size == _items.Length)
        {
            // Удваиваем размер массива, когда стек полон
            Array.Resize(ref _items, _items.Length * 2);
        }
        _items[_size] = item;
        _size++;
    }

    public T Pop()
    {
        if (_size == 0)
        {
            throw new InvalidOperationException("Stack is empty.");
        }
        _size--;
        T item = _items[_size]; // Сохраняем значение перед уменьшением _size
        _items[_size] = default(T); // Освобождаем ссылку, чтобы сборщик мусора мог ее удалить
        return item;
    }

    public T Peek()
    {
        if (_size == 0)
        {
            throw new InvalidOperationException("Stack is empty.");
        }
        return _items[_size - 1];
    }

    public bool IsEmpty()
    {
        return _size == 0;
    }
}

// Пример использования
public class StackExample
{
    public static void Main(string[] args)
    {
        MyStack<int> stack = new MyStack<int>();
        stack.Push(1);
        stack.Push(2);
        stack.Push(3);

        Console.WriteLine("Peek: " + stack.Peek()); // Вывод: Peek: 3
        Console.WriteLine("Pop: " + stack.Pop());   // Вывод: Pop: 3
        Console.WriteLine("Count: " + stack.Count);  // Вывод: Count: 2

        while (!stack.IsEmpty())
        {
            Console.WriteLine("Pop: " + stack.Pop());
        }

        Console.WriteLine("IsEmpty: " + stack.IsEmpty()); // Вывод: IsEmpty: True

        // Попытка Pop() из пустого стека вызовет исключение InvalidOperationException
        // try {
        //     stack.Pop();
        // } catch (InvalidOperationException e) {
        //     Console.WriteLine(e.Message);
        // }
    }
}
```

**Пояснения:**

1.  **`MyStack<T>`:** Это класс стека, который может хранить элементы любого типа `T`.
2.  **`_items`:** Массив, в котором хранятся элементы стека.
3.  **`_size`:**  Количество элементов в стеке.
4.  **`DefaultCapacity`:** Начальная вместимость стека.
5.  **`Push(T item)`:** Добавляет элемент в вершину стека.  Если массив заполнен, он удваивается в размере.
6.  **`Pop()`:** Удаляет и возвращает элемент с вершины стека.  Выбрасывает исключение `InvalidOperationException`, если стек пуст. После извлечения, элемент в массиве устанавливается в значение по умолчанию для типа `T` (`default(T)`) для облегчения работы сборщика мусора.
7.  **`Peek()`:**  Возвращает элемент с вершины стека, не удаляя его. Выбрасывает исключение `InvalidOperationException`, если стек пуст.
8.  **`Count`:**  Возвращает количество элементов в стеке.
9.  **`IsEmpty()`:**  Возвращает `true`, если стек пуст, и `false` в противном случае.
10. **Обработка переполнения стека:**  Когда массив заполняется, размер массива удваивается с помощью `Array.Resize()`.  Это обеспечивает динамический рост стека.
11. **Обработка пустого стека:**  Методы `Pop()` и `Peek()` выбрасывают исключение `InvalidOperationException`, если стек пуст.
12. **Освобождение ссылок:** В методе `Pop()`, после сохранения извлеченного элемента, `_items[_size]` устанавливается в `default(T)`. Это помогает сборщику мусора, освобождая ссылку на объект, который больше не используется в стеке.  Это может быть особенно важно для стеков, хранящих большие объекты.
13. **Конструкторы:** Реализованы два конструктора: один по умолчанию, использующий `DefaultCapacity`, и один позволяющий задать начальную емкость.

Эта реализация стека является потокобезопасной (thread-safe).  Если вам нужна потокобезопасная версия стека, используйте `System.Collections.Concurrent.ConcurrentStack<T>`.
