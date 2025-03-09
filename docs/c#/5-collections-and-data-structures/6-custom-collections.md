# Custom Collections

Bu bölümde, C#'ta özel koleksiyon sınıfları oluşturmayı inceleyeceğiz. Özel koleksiyonlar, standart koleksiyonların sağlamadığı özel davranışlar ve işlevsellikler gerektiren durumlarda kullanışlıdır.

## 1. IEnumerable<T> Implementation

`IEnumerable<T>` arayüzü, bir koleksiyonun üzerinde döngü yapılabilmesini sağlayan en temel arayüzdür. Bu arayüzü uygulamak, koleksiyonunuzun `foreach` döngüsünde kullanılabilmesini sağlar.

### Temel IEnumerable<T> Uygulaması

```csharp
public class SimpleCollection<T> : IEnumerable<T>
{
    private readonly T[] _items;
    
    public SimpleCollection(T[] items)
    {
        _items = items ?? new T[0];
    }
    
    // Generic IEnumerable<T> için gerekli metot
    public IEnumerator<T> GetEnumerator()
    {
        for (int i = 0; i < _items.Length; i++)
        {
            yield return _items[i];
        }
    }
    
    // Non-generic IEnumerable için gerekli metot
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}

// Kullanım
var collection = new SimpleCollection<int>(new[] { 1, 2, 3, 4, 5 });
foreach (int item in collection)
{
    Console.WriteLine(item);
}
```

### Özel Sıralama ile IEnumerable<T>

```csharp
public class ReverseCollection<T> : IEnumerable<T>
{
    private readonly T[] _items;
    
    public ReverseCollection(T[] items)
    {
        _items = items ?? new T[0];
    }
    
    public IEnumerator<T> GetEnumerator()
    {
        // Ters sırada döngü
        for (int i = _items.Length - 1; i >= 0; i--)
        {
            yield return _items[i];
        }
    }
    
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}

// Kullanım
var reverseCollection = new ReverseCollection<string>(new[] { "a", "b", "c" });
foreach (string item in reverseCollection)
{
    Console.WriteLine(item); // "c", "b", "a" sırasında
}
```

### Lazy Evaluation ile IEnumerable<T>

```csharp
public class LazyFilteredCollection<T> : IEnumerable<T>
{
    private readonly IEnumerable<T> _source;
    private readonly Func<T, bool> _predicate;
    
    public LazyFilteredCollection(IEnumerable<T> source, Func<T, bool> predicate)
    {
        _source = source ?? throw new ArgumentNullException(nameof(source));
        _predicate = predicate ?? throw new ArgumentNullException(nameof(predicate));
    }
    
    public IEnumerator<T> GetEnumerator()
    {
        // Lazy evaluation - elemanlar istendiğinde filtrelenir
        foreach (T item in _source)
        {
            if (_predicate(item))
            {
                yield return item;
            }
        }
    }
    
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}

// Kullanım
var numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
var evenNumbers = new LazyFilteredCollection<int>(numbers, n => n % 2 == 0);

foreach (int number in evenNumbers)
{
    Console.WriteLine(number); // 2, 4, 6, 8, 10
}
```

## 2. ICollection<T> Implementation

`ICollection<T>` arayüzü, `IEnumerable<T>`'yi genişleterek koleksiyona eleman ekleme, çıkarma ve sayma gibi temel işlemleri sağlar.

### Temel ICollection<T> Uygulaması

```csharp
public class ObservableCollection<T> : ICollection<T>
{
    private readonly List<T> _items = new List<T>();
    
    // Koleksiyon değişiklik olayları
    public event EventHandler<T> ItemAdded;
    public event EventHandler<T> ItemRemoved;
    
    // ICollection<T> özellikleri
    public int Count => _items.Count;
    public bool IsReadOnly => false;
    
    // ICollection<T> metotları
    public void Add(T item)
    {
        _items.Add(item);
        ItemAdded?.Invoke(this, item);
    }
    
    public void Clear()
    {
        var oldItems = new List<T>(_items);
        _items.Clear();
        
        foreach (T item in oldItems)
        {
            ItemRemoved?.Invoke(this, item);
        }
    }
    
    public bool Contains(T item)
    {
        return _items.Contains(item);
    }
    
    public void CopyTo(T[] array, int arrayIndex)
    {
        _items.CopyTo(array, arrayIndex);
    }
    
    public bool Remove(T item)
    {
        bool removed = _items.Remove(item);
        if (removed)
        {
            ItemRemoved?.Invoke(this, item);
        }
        return removed;
    }
    
    // IEnumerable<T> metotları
    public IEnumerator<T> GetEnumerator()
    {
        return _items.GetEnumerator();
    }
    
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}

// Kullanım
var observableCollection = new ObservableCollection<string>();

observableCollection.ItemAdded += (sender, item) => 
    Console.WriteLine($"Eklendi: {item}");
    
observableCollection.ItemRemoved += (sender, item) => 
    Console.WriteLine($"Çıkarıldı: {item}");

observableCollection.Add("Elma");
observableCollection.Add("Armut");
observableCollection.Remove("Elma");
```

### Sınırlı Kapasiteli Koleksiyon

```csharp
public class BoundedCollection<T> : ICollection<T>
{
    private readonly List<T> _items;
    private readonly int _maxCapacity;
    
    public BoundedCollection(int maxCapacity)
    {
        if (maxCapacity <= 0)
            throw new ArgumentOutOfRangeException(nameof(maxCapacity), "Kapasite pozitif olmalıdır.");
            
        _maxCapacity = maxCapacity;
        _items = new List<T>(maxCapacity);
    }
    
    public int Count => _items.Count;
    public bool IsReadOnly => false;
    public int MaxCapacity => _maxCapacity;
    public bool IsFull => Count >= MaxCapacity;
    
    public void Add(T item)
    {
        if (IsFull)
            throw new InvalidOperationException("Koleksiyon dolu.");
            
        _items.Add(item);
    }
    
    public bool TryAdd(T item)
    {
        if (IsFull)
            return false;
            
        _items.Add(item);
        return true;
    }
    
    public void Clear() => _items.Clear();
    
    public bool Contains(T item) => _items.Contains(item);
    
    public void CopyTo(T[] array, int arrayIndex) => _items.CopyTo(array, arrayIndex);
    
    public bool Remove(T item) => _items.Remove(item);
    
    public IEnumerator<T> GetEnumerator() => _items.GetEnumerator();
    
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() => GetEnumerator();
}

// Kullanım
var boundedCollection = new BoundedCollection<int>(5);

for (int i = 0; i < 5; i++)
{
    boundedCollection.Add(i);
}

// Kapasite aşımı
try
{
    boundedCollection.Add(100); // InvalidOperationException fırlatır
}
catch (InvalidOperationException ex)
{
    Console.WriteLine(ex.Message);
}

// Güvenli ekleme
bool added = boundedCollection.TryAdd(100); // false döner
```

## 3. IList<T> Implementation

`IList<T>` arayüzü, `ICollection<T>`'yi genişleterek indeks tabanlı erişim ve manipülasyon sağlar.

### Temel IList<T> Uygulaması

```csharp
public class CircularList<T> : IList<T>
{
    private readonly List<T> _items = new List<T>();
    
    public int Count => _items.Count;
    public bool IsReadOnly => false;
    
    // Indeksleyici - dairesel erişim sağlar
    public T this[int index]
    {
        get
        {
            if (Count == 0)
                throw new InvalidOperationException("Liste boş.");
                
            // Negatif indeksleri ve taşmaları dairesel olarak ele al
            int actualIndex = ((index % Count) + Count) % Count;
            return _items[actualIndex];
        }
        set
        {
            if (Count == 0)
                throw new InvalidOperationException("Liste boş.");
                
            int actualIndex = ((index % Count) + Count) % Count;
            _items[actualIndex] = value;
        }
    }
    
    public int IndexOf(T item) => _items.IndexOf(item);
    
    public void Insert(int index, T item)
    {
        if (index < 0 || index > Count)
            throw new ArgumentOutOfRangeException(nameof(index));
            
        _items.Insert(index, item);
    }
    
    public void RemoveAt(int index)
    {
        if (index < 0 || index >= Count)
            throw new ArgumentOutOfRangeException(nameof(index));
            
        _items.RemoveAt(index);
    }
    
    public void Add(T item) => _items.Add(item);
    
    public void Clear() => _items.Clear();
    
    public bool Contains(T item) => _items.Contains(item);
    
    public void CopyTo(T[] array, int arrayIndex) => _items.CopyTo(array, arrayIndex);
    
    public bool Remove(T item) => _items.Remove(item);
    
    public IEnumerator<T> GetEnumerator() => _items.GetEnumerator();
    
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() => GetEnumerator();
}

// Kullanım
var circularList = new CircularList<char>();
circularList.Add('A');
circularList.Add('B');
circularList.Add('C');

// Dairesel indeksleme
Console.WriteLine(circularList[0]);  // A
Console.WriteLine(circularList[1]);  // B
Console.WriteLine(circularList[2]);  // C
Console.WriteLine(circularList[3]);  // A (dairesel)
Console.WriteLine(circularList[4]);  // B (dairesel)
Console.WriteLine(circularList[-1]); // C (dairesel, sondan başa)
```

### Sıralı Liste Uygulaması

```csharp
public class SortedList<T> : IList<T> where T : IComparable<T>
{
    private readonly List<T> _items = new List<T>();
    
    public int Count => _items.Count;
    public bool IsReadOnly => false;
    
    public T this[int index]
    {
        get
        {
            if (index < 0 || index >= Count)
                throw new ArgumentOutOfRangeException(nameof(index));
                
            return _items[index];
        }
        set
        {
            if (index < 0 || index >= Count)
                throw new ArgumentOutOfRangeException(nameof(index));
                
            // Mevcut elemanı çıkar
            _items.RemoveAt(index);
            
            // Yeni elemanı sıralı şekilde ekle
            Add(value);
        }
    }
    
    public void Add(T item)
    {
        // Sıralı ekleme için doğru konumu bul
        int insertIndex = 0;
        while (insertIndex < Count && _items[insertIndex].CompareTo(item) < 0)
        {
            insertIndex++;
        }
        
        _items.Insert(insertIndex, item);
    }
    
    public int IndexOf(T item) => _items.IndexOf(item);
    
    public void Insert(int index, T item)
    {
        // Insert metodu sıralamayı bozabileceği için Add metodunu kullan
        Add(item);
    }
    
    public void RemoveAt(int index)
    {
        if (index < 0 || index >= Count)
            throw new ArgumentOutOfRangeException(nameof(index));
            
        _items.RemoveAt(index);
    }
    
    public void Clear() => _items.Clear();
    
    public bool Contains(T item) => _items.Contains(item);
    
    public void CopyTo(T[] array, int arrayIndex) => _items.CopyTo(array, arrayIndex);
    
    public bool Remove(T item) => _items.Remove(item);
    
    public IEnumerator<T> GetEnumerator() => _items.GetEnumerator();
    
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() => GetEnumerator();
}

// Kullanım
var sortedList = new SortedList<int>();
sortedList.Add(5);
sortedList.Add(2);
sortedList.Add(8);
sortedList.Add(1);

// Elemanlar sıralı olarak saklanır
foreach (int item in sortedList)
{
    Console.WriteLine(item); // 1, 2, 5, 8
}
```

## 4. Custom Collection Serialization

Özel koleksiyonların serileştirilmesi, koleksiyonun durumunu kalıcı depolama veya ağ üzerinden aktarım için dönüştürmeyi içerir.

### ISerializable Uygulaması

```csharp
[Serializable]
public class SerializableCollection<T> : ICollection<T>, ISerializable where T : ISerializable
{
    private readonly List<T> _items = new List<T>();
    
    // Standart constructor
    public SerializableCollection() { }
    
    // Deserialization constructor
    protected SerializableCollection(SerializationInfo info, StreamingContext context)
    {
        int count = info.GetInt32("Count");
        for (int i = 0; i < count; i++)
        {
            T item = (T)info.GetValue($"Item_{i}", typeof(T));
            _items.Add(item);
        }
    }
    
    // ISerializable implementation
    public void GetObjectData(SerializationInfo info, StreamingContext context)
    {
        info.AddValue("Count", Count);
        for (int i = 0; i < Count; i++)
        {
            info.AddValue($"Item_{i}", _items[i]);
        }
    }
    
    // ICollection<T> implementation
    public int Count => _items.Count;
    public bool IsReadOnly => false;
    public void Add(T item) => _items.Add(item);
    public void Clear() => _items.Clear();
    public bool Contains(T item) => _items.Contains(item);
    public void CopyTo(T[] array, int arrayIndex) => _items.CopyTo(array, arrayIndex);
    public bool Remove(T item) => _items.Remove(item);
    public IEnumerator<T> GetEnumerator() => _items.GetEnumerator();
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() => GetEnumerator();
}
```

### JSON Serileştirme

```csharp
public class JsonSerializableCollection<T> : ICollection<T>
{
    private readonly List<T> _items = new List<T>();
    
    // ICollection<T> implementation
    public int Count => _items.Count;
    public bool IsReadOnly => false;
    public void Add(T item) => _items.Add(item);
    public void Clear() => _items.Clear();
    public bool Contains(T item) => _items.Contains(item);
    public void CopyTo(T[] array, int arrayIndex) => _items.CopyTo(array, arrayIndex);
    public bool Remove(T item) => _items.Remove(item);
    public IEnumerator<T> GetEnumerator() => _items.GetEnumerator();
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() => GetEnumerator();
    
    // JSON serileştirme metotları
    public string ToJson()
    {
        return System.Text.Json.JsonSerializer.Serialize(_items);
    }
    
    public static JsonSerializableCollection<T> FromJson(string json)
    {
        var collection = new JsonSerializableCollection<T>();
        var items = System.Text.Json.JsonSerializer.Deserialize<List<T>>(json);
        
        if (items != null)
        {
            foreach (T item in items)
            {
                collection.Add(item);
            }
        }
        
        return collection;
    }
    
    // Dosyaya kaydetme ve yükleme
    public void SaveToFile(string filePath)
    {
        string json = ToJson();
        File.WriteAllText(filePath, json);
    }
    
    public static JsonSerializableCollection<T> LoadFromFile(string filePath)
    {
        if (!File.Exists(filePath))
            throw new FileNotFoundException("Dosya bulunamadı.", filePath);
            
        string json = File.ReadAllText(filePath);
        return FromJson(json);
    }
}

// Kullanım
var collection = new JsonSerializableCollection<Person>();
collection.Add(new Person { Name = "Ali", Age = 30 });
collection.Add(new Person { Name = "Ayşe", Age = 25 });

// JSON'a dönüştürme
string json = collection.ToJson();
Console.WriteLine(json);

// Dosyaya kaydetme
collection.SaveToFile("people.json");

// Dosyadan yükleme
var loadedCollection = JsonSerializableCollection<Person>.LoadFromFile("people.json");
```

## 5. Collection Change Notification

Koleksiyon değişiklik bildirimleri, koleksiyondaki değişiklikleri dinlemek ve bu değişikliklere tepki vermek için kullanılır.

### INotifyCollectionChanged Uygulaması

```csharp
public class ObservableList<T> : IList<T>, INotifyCollectionChanged, INotifyPropertyChanged
{
    private readonly List<T> _items = new List<T>();
    
    // INotifyCollectionChanged implementation
    public event NotifyCollectionChangedEventHandler CollectionChanged;
    
    // INotifyPropertyChanged implementation
    public event PropertyChangedEventHandler PropertyChanged;
    
    // IList<T> implementation
    public int Count => _items.Count;
    public bool IsReadOnly => false;
    
    public T this[int index]
    {
        get => _items[index];
        set
        {
            T oldItem = _items[index];
            _items[index] = value;
            
            OnPropertyChanged(nameof(this[index]));
            OnCollectionChanged(NotifyCollectionChangedAction.Replace, value, oldItem, index);
        }
    }
    
    public void Add(T item)
    {
        _items.Add(item);
        
        OnPropertyChanged(nameof(Count));
        OnCollectionChanged(NotifyCollectionChangedAction.Add, item, _items.Count - 1);
    }
    
    public void Clear()
    {
        var oldItems = new List<T>(_items);
        _items.Clear();
        
        OnPropertyChanged(nameof(Count));
        OnCollectionChanged(NotifyCollectionChangedAction.Reset, oldItems);
    }
    
    public bool Contains(T item) => _items.Contains(item);
    
    public void CopyTo(T[] array, int arrayIndex) => _items.CopyTo(array, arrayIndex);
    
    public int IndexOf(T item) => _items.IndexOf(item);
    
    public void Insert(int index, T item)
    {
        _items.Insert(index, item);
        
        OnPropertyChanged(nameof(Count));
        OnCollectionChanged(NotifyCollectionChangedAction.Add, item, index);
    }
    
    public bool Remove(T item)
    {
        int index = _items.IndexOf(item);
        if (index < 0)
            return false;
            
        _items.RemoveAt(index);
        
        OnPropertyChanged(nameof(Count));
        OnCollectionChanged(NotifyCollectionChangedAction.Remove, item, index);
        
        return true;
    }
    
    public void RemoveAt(int index)
    {
        T removedItem = _items[index];
        _items.RemoveAt(index);
        
        OnPropertyChanged(nameof(Count));
        OnCollectionChanged(NotifyCollectionChangedAction.Remove, removedItem, index);
    }
    
    public IEnumerator<T> GetEnumerator() => _items.GetEnumerator();
    
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() => GetEnumerator();
    
    // Helper methods
    protected virtual void OnCollectionChanged(NotifyCollectionChangedAction action, object item, int index)
    {
        CollectionChanged?.Invoke(this, new NotifyCollectionChangedEventArgs(action, item, index));
    }
    
    protected virtual void OnCollectionChanged(NotifyCollectionChangedAction action, object newItem, object oldItem, int index)
    {
        CollectionChanged?.Invoke(this, new NotifyCollectionChangedEventArgs(action, newItem, oldItem, index));
    }
    
    protected virtual void OnCollectionChanged(NotifyCollectionChangedAction action, IList items)
    {
        CollectionChanged?.Invoke(this, new NotifyCollectionChangedEventArgs(action, items));
    }
    
    protected virtual void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}

// Kullanım
var observableList = new ObservableList<string>();

observableList.CollectionChanged += (sender, e) =>
{
    switch (e.Action)
    {
        case NotifyCollectionChangedAction.Add:
            Console.WriteLine($"Eklendi: {e.NewItems[0]} (İndeks: {e.NewStartingIndex})");
            break;
        case NotifyCollectionChangedAction.Remove:
            Console.WriteLine($"Çıkarıldı: {e.OldItems[0]} (İndeks: {e.OldStartingIndex})");
            break;
        case NotifyCollectionChangedAction.Replace:
            Console.WriteLine($"Değiştirildi: {e.OldItems[0]} -> {e.NewItems[0]} (İndeks: {e.NewStartingIndex})");
            break;
        case NotifyCollectionChangedAction.Reset:
            Console.WriteLine("Koleksiyon sıfırlandı");
            break;
    }
};

observableList.Add("Elma");
observableList.Add("Armut");
observableList[0] = "Kiraz";
observableList.Remove("Armut");
observableList.Clear();
```

## 6. Custom Collection Validation

Özel koleksiyon doğrulama, koleksiyona eklenen öğelerin belirli kurallara uymasını sağlar.

### Doğrulama ile Koleksiyon

```csharp
public class ValidatingCollection<T> : ICollection<T>
{
    private readonly List<T> _items = new List<T>();
    private readonly Func<T, bool> _validator;
    
    public ValidatingCollection(Func<T, bool> validator)
    {
        _validator = validator ?? throw new ArgumentNullException(nameof(validator));
    }
    
    public int Count => _items.Count;
    public bool IsReadOnly => false;
    
    public void Add(T item)
    {
        if (!_validator(item))
            throw new ArgumentException("Öğe doğrulama kurallarını karşılamıyor.");
            
        _items.Add(item);
    }
    
    public bool TryAdd(T item)
    {
        if (!_validator(item))
            return false;
            
        _items.Add(item);
        return true;
    }
    
    public void Clear() => _items.Clear();
    
    public bool Contains(T item) => _items.Contains(item);
    
    public void CopyTo(T[] array, int arrayIndex) => _items.CopyTo(array, arrayIndex);
    
    public bool Remove(T item) => _items.Remove(item);
    
    public IEnumerator<T> GetEnumerator() => _items.GetEnumerator();
    
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() => GetEnumerator();
}

// Kullanım
// Sadece pozitif sayıları kabul eden koleksiyon
var positiveNumbers = new ValidatingCollection<int>(n => n > 0);

positiveNumbers.Add(5);  // Başarılı
positiveNumbers.Add(10); // Başarılı

try
{
    positiveNumbers.Add(-3); // ArgumentException fırlatır
}
catch (ArgumentException ex)
{
    Console.WriteLine(ex.Message);
}

// Güvenli ekleme
bool added = positiveNumbers.TryAdd(-5); // false döner, exception fırlatmaz
```

### Tip Güvenli Heterojen Koleksiyon

```csharp
public class TypeSafeHeterogenousCollection : IEnumerable
{
    private readonly Dictionary<Type, object> _items = new Dictionary<Type, object>();
    
    public void Add<T>(T item)
    {
        Type type = typeof(T);
        
        if (_items.ContainsKey(type))
            throw new ArgumentException($"Bu türde bir öğe zaten var: {type.Name}");
            
        _items[type] = item;
    }
    
    public T Get<T>()
    {
        Type type = typeof(T);
        
        if (!_items.TryGetValue(type, out object item))
            throw new KeyNotFoundException($"Bu türde bir öğe bulunamadı: {type.Name}");
            
        return (T)item;
    }
    
    public bool TryGet<T>(out T value)
    {
        Type type = typeof(T);
        
        if (_items.TryGetValue(type, out object item))
        {
            value = (T)item;
            return true;
        }
        
        value = default;
        return false;
    }
    
    public bool Contains<T>() => _items.ContainsKey(typeof(T));
    
    public bool Remove<T>() => _items.Remove(typeof(T));
    
    public void Clear() => _items.Clear();
    
    public int Count => _items.Count;
    
    public IEnumerator GetEnumerator() => _items.Values.GetEnumerator();
}

// Kullanım
var heterogenousCollection = new TypeSafeHeterogenousCollection();

heterogenousCollection.Add("Bir string");
heterogenousCollection.Add(42);
heterogenousCollection.Add(new DateTime(2023, 1, 1));

string stringValue = heterogenousCollection.Get<string>();
int intValue = heterogenousCollection.Get<int>();
DateTime dateValue = heterogenousCollection.Get<DateTime>();

Console.WriteLine($"String: {stringValue}");
Console.WriteLine($"Int: {intValue}");
Console.WriteLine($"Date: {dateValue}");
```

Özel koleksiyonlar, uygulamanızın özel gereksinimlerini karşılamak için standart koleksiyonların ötesinde işlevsellik sağlar. Doğru arayüzleri uygulamak ve gerekli davranışları eklemek, veri yönetimini daha esnek ve güçlü hale getirebilir. 