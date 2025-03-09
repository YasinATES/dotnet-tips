# Queue ve Stack

Bu bölümde, C#'ın önemli veri yapılarından olan `Queue<T>` (Kuyruk) ve `Stack<T>` (Yığın) koleksiyonlarını inceleyeceğiz. Bu veri yapıları, verilerin belirli bir düzende eklenip çıkarılmasını gerektiren senaryolarda kullanılır.

## 1. Queue<T> FIFO Operations

`Queue<T>`, FIFO (First-In-First-Out) prensibiyle çalışan bir veri yapısıdır. Bu, kuyruğa ilk eklenen elemanın ilk çıkarılacağı anlamına gelir - tıpkı gerçek hayattaki bir kuyruk gibi.

### Temel Queue Operasyonları

```csharp
// Queue oluşturma
Queue<string> customerQueue = new Queue<string>();

// Kuyruğa eleman ekleme (Enqueue)
customerQueue.Enqueue("Müşteri 1");
customerQueue.Enqueue("Müşteri 2");
customerQueue.Enqueue("Müşteri 3");

// Kuyruktan eleman çıkarma (Dequeue)
string nextCustomer = customerQueue.Dequeue(); // "Müşteri 1"
Console.WriteLine($"Sıradaki müşteri: {nextCustomer}");

// Kuyruğun başındaki elemanı görüntüleme (Peek)
string currentCustomer = customerQueue.Peek(); // "Müşteri 2"
Console.WriteLine($"Şu anki müşteri: {currentCustomer}");

// Kuyrukta eleman arama
bool containsCustomer3 = customerQueue.Contains("Müşteri 3"); // true

// Kuyruk boyutu
int count = customerQueue.Count; // 2

// Kuyruğu temizleme
customerQueue.Clear();
```

### Queue İterasyonu

```csharp
Queue<int> numbers = new Queue<int>();
numbers.Enqueue(10);
numbers.Enqueue(20);
numbers.Enqueue(30);
numbers.Enqueue(40);

// Foreach ile döngü
foreach (int number in numbers)
{
    Console.WriteLine(number);
}

// Diziye dönüştürme ve işlem yapma
int[] array = numbers.ToArray();
Array.Reverse(array);

// Kuyruğu kopyalama
Queue<int> copy = new Queue<int>(numbers);
```

### Queue Kullanım Senaryoları

1. **İşlem Sırası Yönetimi**
   ```csharp
   Queue<PrintJob> printQueue = new Queue<PrintJob>();
   
   // Yazdırma işlerini kuyruğa ekleme
   printQueue.Enqueue(new PrintJob("Rapor.pdf", 5));
   printQueue.Enqueue(new PrintJob("Fatura.pdf", 2));
   
   // Yazdırma işlemlerini sırayla işleme
   while (printQueue.Count > 0)
   {
       PrintJob job = printQueue.Dequeue();
       Console.WriteLine($"Yazdırılıyor: {job.FileName}, {job.Pages} sayfa");
       // Yazdırma işlemi...
   }
   ```

2. **Breadth-First Search (BFS) Algoritması**
   ```csharp
   public void BreadthFirstSearch(Node root)
   {
       if (root == null) return;
       
       Queue<Node> queue = new Queue<Node>();
       HashSet<Node> visited = new HashSet<Node>();
       
       queue.Enqueue(root);
       visited.Add(root);
       
       while (queue.Count > 0)
       {
           Node current = queue.Dequeue();
           Console.WriteLine(current.Value);
           
           foreach (Node neighbor in current.Neighbors)
           {
               if (!visited.Contains(neighbor))
               {
                   visited.Add(neighbor);
                   queue.Enqueue(neighbor);
               }
           }
       }
   }
   ```

3. **Mesaj İşleme Sistemleri**
   ```csharp
   Queue<Message> messageQueue = new Queue<Message>();
   
   // Mesaj üretici
   void ProduceMessages()
   {
       while (true)
       {
           Message message = GenerateMessage();
           lock (messageQueue)
           {
               messageQueue.Enqueue(message);
           }
           Thread.Sleep(100); // Mesaj üretme hızını sınırla
       }
   }
   
   // Mesaj tüketici
   void ConsumeMessages()
   {
       while (true)
       {
           Message message = null;
           lock (messageQueue)
           {
               if (messageQueue.Count > 0)
               {
                   message = messageQueue.Dequeue();
               }
           }
           
           if (message != null)
           {
               ProcessMessage(message);
           }
           else
           {
               Thread.Sleep(10); // Kuyruk boşsa bekle
           }
       }
   }
   ```

## 2. Stack<T> LIFO Operations

`Stack<T>`, LIFO (Last-In-First-Out) prensibiyle çalışan bir veri yapısıdır. Bu, yığına en son eklenen elemanın ilk çıkarılacağı anlamına gelir - tıpkı üst üste dizilmiş tabaklar gibi.

### Temel Stack Operasyonları

```csharp
// Stack oluşturma
Stack<string> browserHistory = new Stack<string>();

// Yığına eleman ekleme (Push)
browserHistory.Push("https://www.google.com");
browserHistory.Push("https://www.wikipedia.org");
browserHistory.Push("https://www.github.com");

// Yığından eleman çıkarma (Pop)
string lastVisited = browserHistory.Pop(); // "https://www.github.com"
Console.WriteLine($"Son ziyaret edilen: {lastVisited}");

// Yığının üstündeki elemanı görüntüleme (Peek)
string currentPage = browserHistory.Peek(); // "https://www.wikipedia.org"
Console.WriteLine($"Şu anki sayfa: {currentPage}");

// Yığında eleman arama
bool containsGoogle = browserHistory.Contains("https://www.google.com"); // true

// Yığın boyutu
int count = browserHistory.Count; // 2

// Yığını temizleme
browserHistory.Clear();
```

### Stack İterasyonu

```csharp
Stack<int> numbers = new Stack<int>();
numbers.Push(10);
numbers.Push(20);
numbers.Push(30);
numbers.Push(40);

// Foreach ile döngü (LIFO sırasında)
foreach (int number in numbers)
{
    Console.WriteLine(number); // 40, 30, 20, 10 sırasında
}

// Diziye dönüştürme
int[] array = numbers.ToArray(); // [40, 30, 20, 10]

// Yığını kopyalama
Stack<int> copy = new Stack<int>(numbers);
```

### Stack Kullanım Senaryoları

1. **Geri Al İşlemi**
   ```csharp
   Stack<ICommand> undoStack = new Stack<ICommand>();
   Stack<ICommand> redoStack = new Stack<ICommand>();
   
   void ExecuteCommand(ICommand command)
   {
       command.Execute();
       undoStack.Push(command);
       redoStack.Clear(); // Yeni bir komut yürütüldüğünde redo stack'i temizle
   }
   
   void Undo()
   {
       if (undoStack.Count > 0)
       {
           ICommand command = undoStack.Pop();
           command.Undo();
           redoStack.Push(command);
       }
   }
   
   void Redo()
   {
       if (redoStack.Count > 0)
       {
           ICommand command = redoStack.Pop();
           command.Execute();
           undoStack.Push(command);
       }
   }
   ```

2. **İfade Değerlendirme**
   ```csharp
   public int EvaluatePostfix(string expression)
   {
       Stack<int> stack = new Stack<int>();
       string[] tokens = expression.Split(' ');
       
       foreach (string token in tokens)
       {
           if (int.TryParse(token, out int number))
           {
               stack.Push(number);
           }
           else
           {
               int b = stack.Pop();
               int a = stack.Pop();
               
               switch (token)
               {
                   case "+": stack.Push(a + b); break;
                   case "-": stack.Push(a - b); break;
                   case "*": stack.Push(a * b); break;
                   case "/": stack.Push(a / b); break;
               }
           }
       }
       
       return stack.Pop();
   }
   
   // Kullanım
   int result = EvaluatePostfix("3 4 + 2 * 7 /"); // (3 + 4) * 2 / 7 = 2
   ```

3. **Derinlik Öncelikli Arama (DFS)**
   ```csharp
   public void DepthFirstSearch(Node root)
   {
       if (root == null) return;
       
       Stack<Node> stack = new Stack<Node>();
       HashSet<Node> visited = new HashSet<Node>();
       
       stack.Push(root);
       
       while (stack.Count > 0)
       {
           Node current = stack.Pop();
           
           if (!visited.Contains(current))
           {
               Console.WriteLine(current.Value);
               visited.Add(current);
               
               // Komşuları ters sırada ekle (doğal DFS sırası için)
               foreach (Node neighbor in current.Neighbors.Reverse())
               {
                   if (!visited.Contains(neighbor))
                   {
                       stack.Push(neighbor);
                   }
               }
           }
       }
   }
   ```

## 3. Circular Buffer Pattern

Dairesel tampon (Circular Buffer), sabit boyutlu bir veri yapısıdır ve FIFO mantığıyla çalışır. Tampon dolduğunda, yeni elemanlar en eski elemanların üzerine yazılır. Bu yapı, bellek kullanımını optimize etmek için kullanılır.

### Circular Buffer Implementasyonu

```csharp
public class CircularBuffer<T>
{
    private readonly T[] _buffer;
    private int _start;
    private int _end;
    private int _count;
    
    public CircularBuffer(int capacity)
    {
        _buffer = new T[capacity];
        _start = 0;
        _end = 0;
        _count = 0;
    }
    
    public int Capacity => _buffer.Length;
    
    public int Count => _count;
    
    public bool IsFull => Count == Capacity;
    
    public bool IsEmpty => Count == 0;
    
    public void Enqueue(T item)
    {
        if (IsFull)
        {
            // Tampon dolu, en eski elemanın üzerine yaz
            _buffer[_end] = item;
            _end = (_end + 1) % Capacity;
            _start = (_start + 1) % Capacity; // Start'ı da ilerlet
        }
        else
        {
            // Normal ekleme
            _buffer[_end] = item;
            _end = (_end + 1) % Capacity;
            _count++;
        }
    }
    
    public T Dequeue()
    {
        if (IsEmpty)
            throw new InvalidOperationException("Buffer is empty");
        
        T item = _buffer[_start];
        _start = (_start + 1) % Capacity;
        _count--;
        
        return item;
    }
    
    public T Peek()
    {
        if (IsEmpty)
            throw new InvalidOperationException("Buffer is empty");
        
        return _buffer[_start];
    }
    
    public void Clear()
    {
        _start = 0;
        _end = 0;
        _count = 0;
        Array.Clear(_buffer, 0, _buffer.Length);
    }
    
    public IEnumerator<T> GetEnumerator()
    {
        if (IsEmpty) yield break;
        
        int currentIndex = _start;
        int itemsReturned = 0;
        
        while (itemsReturned < _count)
        {
            yield return _buffer[currentIndex];
            currentIndex = (currentIndex + 1) % Capacity;
            itemsReturned++;
        }
    }
}
```

### Circular Buffer Kullanımı

```csharp
// Dairesel tampon oluşturma
CircularBuffer<int> buffer = new CircularBuffer<int>(3);

// Eleman ekleme
buffer.Enqueue(1);
buffer.Enqueue(2);
buffer.Enqueue(3);
Console.WriteLine($"Tampon dolu mu: {buffer.IsFull}"); // true

// Tampon doluyken ekleme (en eski eleman üzerine yazılır)
buffer.Enqueue(4);

// Elemanları çıkarma
int first = buffer.Dequeue(); // 2
int second = buffer.Dequeue(); // 3
int third = buffer.Dequeue(); // 4
Console.WriteLine($"Tampon boş mu: {buffer.IsEmpty}"); // true
```

### Circular Buffer Kullanım Senaryoları

1. **Veri Akışı İşleme**
   - Sensör verilerinin geçici depolanması
   - Log kayıtlarının tamponlanması
   - Ağ paketlerinin işlenmesi

2. **Sınırlı Geçmiş Takibi**
   - Son N işlemin kaydedilmesi
   - Sınırlı sayıda geri alma işlemi
   - Kayan pencere algoritmaları

## 4. Priority Queue Implementation

Öncelik kuyruğu (Priority Queue), elemanların öncelik değerlerine göre sıralandığı bir kuyruk türüdür. En yüksek (veya en düşük) önceliğe sahip eleman her zaman ilk çıkarılır.

### Priority Queue Implementasyonu

```csharp
public class PriorityQueue<T> where T : IComparable<T>
{
    private readonly List<T> _heap;
    
    public PriorityQueue()
    {
        _heap = new List<T>();
    }
    
    public int Count => _heap.Count;
    
    public bool IsEmpty => Count == 0;
    
    public void Enqueue(T item)
    {
        _heap.Add(item);
        int childIndex = _heap.Count - 1;
        
        // Heap özelliğini korumak için yukarı doğru düzenleme
        while (childIndex > 0)
        {
            int parentIndex = (childIndex - 1) / 2;
            
            if (_heap[childIndex].CompareTo(_heap[parentIndex]) >= 0)
                break;
                
            // Ebeveyn ve çocuğu değiştir
            T temp = _heap[childIndex];
            _heap[childIndex] = _heap[parentIndex];
            _heap[parentIndex] = temp;
            
            childIndex = parentIndex;
        }
    }
    
    public T Dequeue()
    {
        if (IsEmpty)
            throw new InvalidOperationException("Queue is empty");
            
        T result = _heap[0];
        
        // Son elemanı kök olarak taşı ve heap özelliğini korumak için aşağı doğru düzenleme
        int lastIndex = _heap.Count - 1;
        _heap[0] = _heap[lastIndex];
        _heap.RemoveAt(lastIndex);
        
        lastIndex--;
        
        if (lastIndex > 0)
        {
            int parentIndex = 0;
            
            while (true)
            {
                int leftChildIndex = parentIndex * 2 + 1;
                
                if (leftChildIndex > lastIndex)
                    break;
                    
                int rightChildIndex = leftChildIndex + 1;
                int minIndex = leftChildIndex;
                
                if (rightChildIndex <= lastIndex && _heap[rightChildIndex].CompareTo(_heap[leftChildIndex]) < 0)
                    minIndex = rightChildIndex;
                    
                if (_heap[parentIndex].CompareTo(_heap[minIndex]) <= 0)
                    break;
                    
                // Ebeveyn ve en küçük çocuğu değiştir
                T temp = _heap[parentIndex];
                _heap[parentIndex] = _heap[minIndex];
                _heap[minIndex] = temp;
                
                parentIndex = minIndex;
            }
        }
        
        return result;
    }
    
    public T Peek()
    {
        if (IsEmpty)
            throw new InvalidOperationException("Queue is empty");
            
        return _heap[0];
    }
    
    public void Clear()
    {
        _heap.Clear();
    }
}
```

### Priority Queue Kullanımı

```csharp
// Öncelik kuyruğu oluşturma
PriorityQueue<int> priorityQueue = new PriorityQueue<int>();

// Eleman ekleme
priorityQueue.Enqueue(5);
priorityQueue.Enqueue(2);
priorityQueue.Enqueue(8);
priorityQueue.Enqueue(1);
priorityQueue.Enqueue(10);

// Elemanları çıkarma (öncelik sırasına göre)
while (!priorityQueue.IsEmpty)
{
    Console.WriteLine(priorityQueue.Dequeue()); // 1, 2, 5, 8, 10 sırasında
}
```

### .NET 6 Priority Queue

.NET 6 ile birlikte, `System.Collections.Generic` namespace'inde yerleşik bir `PriorityQueue<TElement, TPriority>` sınıfı eklenmiştir:

```csharp
// .NET 6 PriorityQueue kullanımı
PriorityQueue<string, int> tasks = new PriorityQueue<string, int>();

// Eleman ve öncelik değeri ekleme (düşük değer = yüksek öncelik)
tasks.Enqueue("Acil görev", 1);
tasks.Enqueue("Normal görev", 2);
tasks.Enqueue("Düşük öncelikli görev", 3);
tasks.Enqueue("Çok acil görev", 0);

// Elemanları çıkarma
while (tasks.TryDequeue(out string task, out int priority))
{
    Console.WriteLine($"Görev: {task}, Öncelik: {priority}");
}
// Çıktı:
// Görev: Çok acil görev, Öncelik: 0
// Görev: Acil görev, Öncelik: 1
// Görev: Normal görev, Öncelik: 2
// Görev: Düşük öncelikli görev, Öncelik: 3
```

## 5. Thread-Safe Queue Operations

Çoklu iş parçacığı ortamında, standart `Queue<T>` ve `Stack<T>` sınıfları thread-safe değildir. Bu durumda, `System.Collections.Concurrent` namespace'indeki thread-safe koleksiyonları veya senkronizasyon mekanizmalarını kullanmak gerekir.

### ConcurrentQueue<T> Kullanımı

```csharp
using System.Collections.Concurrent;

// Thread-safe kuyruk oluşturma
ConcurrentQueue<string> messageQueue = new ConcurrentQueue<string>();

// Üretici iş parçacığı
Task producer = Task.Run(() =>
{
    for (int i = 0; i < 100; i++)
    {
        messageQueue.Enqueue($"Mesaj {i}");
        Thread.Sleep(10);
    }
});

// Tüketici iş parçacıkları
Task[] consumers = new Task[3];
for (int i = 0; i < consumers.Length; i++)
{
    int consumerId = i;
    consumers[i] = Task.Run(() =>
    {
        while (!messageQueue.IsEmpty || !producer.IsCompleted)
        {
            if (messageQueue.TryDequeue(out string message))
            {
                Console.WriteLine($"Tüketici {consumerId}: {message}");
            }
            else
            {
                Thread.Sleep(1);
            }
        }
    });
}

// Tüm iş parçacıklarının tamamlanmasını bekle
Task.WaitAll(producer);
Task.WaitAll(consumers);
```

### BlockingCollection<T> Kullanımı

`BlockingCollection<T>`, üretici-tüketici senaryoları için daha yüksek seviyeli bir API sağlar:

```csharp
using System.Collections.Concurrent;

// Sınırlı kapasiteli blocking collection oluşturma
BlockingCollection<int> blockingQueue = new BlockingCollection<int>(boundedCapacity: 100);

// Üretici iş parçacığı
Task producer = Task.Run(() =>
{
    for (int i = 0; i < 500; i++)
    {
        blockingQueue.Add(i); // Kuyruk doluysa bloklar
        Console.WriteLine($"Üretildi: {i}");
    }
    blockingQueue.CompleteAdding(); // Ekleme işleminin bittiğini bildir
});

// Tüketici iş parçacıkları
Task[] consumers = new Task[3];
for (int i = 0; i < consumers.Length; i++)
{
    int consumerId = i;
    consumers[i] = Task.Run(() =>
    {
        foreach (int item in blockingQueue.GetConsumingEnumerable())
        {
            Console.WriteLine($"Tüketici {consumerId}: {item}");
            Thread.Sleep(10); // Tüketim işlemini yavaşlat
        }
    });
}

// Tüm iş parçacıklarının tamamlanmasını bekle
Task.WaitAll(producer);
Task.WaitAll(consumers);
```

## 6. Custom Queue Implementations

Özel gereksinimler için, kendi kuyruk implementasyonlarınızı oluşturabilirsiniz. Aşağıda, bağlı liste tabanlı bir kuyruk örneği verilmiştir:

### Linked List Based Queue

```csharp
public class LinkedListQueue<T>
{
    private class Node
    {
        public T Value { get; }
        public Node Next { get; set; }
        
        public Node(T value)
        {
            Value = value;
            Next = null;
        }
    }
    
    private Node _head;
    private Node _tail;
    private int _count;
    
    public int Count => _count;
    
    public bool IsEmpty => _count == 0;
    
    public void Enqueue(T item)
    {
        Node newNode = new Node(item);
        
        if (_tail == null)
        {
            _head = newNode;
            _tail = newNode;
        }
        else
        {
            _tail.Next = newNode;
            _tail = newNode;
        }
        
        _count++;
    }
    
    public T Dequeue()
    {
        if (IsEmpty)
            throw new InvalidOperationException("Queue is empty");
            
        T value = _head.Value;
        _head = _head.Next;
        
        if (_head == null)
            _tail = null;
            
        _count--;
        
        return value;
    }
    
    public T Peek()
    {
        if (IsEmpty)
            throw new InvalidOperationException("Queue is empty");
            
        return _head.Value;
    }
    
    public void Clear()
    {
        _head = null;
        _tail = null;
        _count = 0;
    }
}
```

### Özel Kuyruk Kullanım Senaryoları

1. **Özel Öncelik Mantığı**
   - İş önceliği ve zaman damgası kombinasyonu
   - Çoklu öncelik seviyesi
   - Dinamik öncelik değişimi

2. **Sınırlı Kuyruk**
   - Maksimum boyut sınırlaması
   - Eski elemanların otomatik silinmesi
   - Kuyruk dolduğunda özel davranış

3. **İzlenebilir Kuyruk**
   - Eleman ekleme/çıkarma olayları
   - Kuyruk durumu izleme
   - Performans metrikleri toplama

Queue ve Stack veri yapıları, belirli senaryolarda veri yönetimi için güçlü araçlardır. Doğru veri yapısını seçmek ve gerektiğinde özelleştirmek, uygulamanızın performansını ve kullanılabilirliğini önemli ölçüde artırabilir. 