# ADO.NET ve Low-Level Veri Erişimi

ADO.NET, .NET platformunda veritabanı işlemleri için kullanılan düşük seviyeli bir veri erişim teknolojisidir. ORM araçlarının aksine, ADO.NET ile veritabanı işlemlerini daha detaylı kontrol edebilir ve performans açısından optimize edebilirsiniz.

## Connection ve Command Nesneleri

ADO.NET'in temel yapı taşları, veritabanı bağlantısını yöneten `Connection` ve SQL komutlarını yürüten `Command` nesneleridir.

### Connection Nesnesi

Connection nesnesi, veritabanı ile uygulama arasındaki bağlantıyı temsil eder.

```csharp
using System;
using System.Data;
using System.Data.SqlClient;

public class VeriTabaniBaglantisi
{
    public void BaglantiyiGoster()
    {
        // Bağlantı dizesi
        string connectionString = "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;";
        
        // SqlConnection nesnesi oluşturma
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            try
            {
                // Bağlantıyı aç
                connection.Open();
                
                // Bağlantı durumunu kontrol et
                if (connection.State == ConnectionState.Open)
                {
                    Console.WriteLine("Veritabanı bağlantısı başarıyla açıldı.");
                    Console.WriteLine($"Sunucu Sürümü: {connection.ServerVersion}");
                    Console.WriteLine($"Veritabanı: {connection.Database}");
                    Console.WriteLine($"Veri Kaynağı: {connection.DataSource}");
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Bağlantı hatası: {ex.Message}");
            }
            finally
            {
                // Bağlantı açıksa kapat
                if (connection.State == ConnectionState.Open)
                {
                    connection.Close();
                    Console.WriteLine("Bağlantı kapatıldı.");
                }
            }
        } // using bloğu sonunda bağlantı otomatik olarak kapatılır
    }
}
```

### Command Nesnesi

Command nesnesi, veritabanında çalıştırılacak SQL sorgularını veya saklı prosedürleri temsil eder.

```csharp
public class VeriTabaniKomutlari
{
    private readonly string connectionString = "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;";
    
    public void KomutCalistir()
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            // SQL sorgusu
            string sql = "UPDATE Products SET UnitPrice = UnitPrice * 1.10 WHERE CategoryID = 1";
            
            // Command nesnesi oluşturma
            using (SqlCommand command = new SqlCommand(sql, connection))
            {
                try
                {
                    connection.Open();
                    
                    // Komutu çalıştır ve etkilenen satır sayısını al
                    int affectedRows = command.ExecuteNonQuery();
                    
                    Console.WriteLine($"Güncellenen ürün sayısı: {affectedRows}");
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Komut hatası: {ex.Message}");
                }
            }
        }
    }
    
    public void SakliProsedurCalistir()
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            // Saklı prosedür adı
            using (SqlCommand command = new SqlCommand("GetProductsByCategory", connection))
            {
                // Komut tipini saklı prosedür olarak ayarla
                command.CommandType = CommandType.StoredProcedure;
                
                // Parametre ekle
                command.Parameters.AddWithValue("@CategoryID", 1);
                
                try
                {
                    connection.Open();
                    
                    // Saklı prosedürden dönen değeri al
                    object result = command.ExecuteScalar();
                    
                    Console.WriteLine($"Sonuç: {result}");
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Saklı prosedür hatası: {ex.Message}");
                }
            }
        }
    }
}
```

## DataReader Kullanımı

DataReader, veritabanından gelen verileri ileri yönlü, salt okunur bir akış olarak sunar. Büyük veri kümeleri için bellek kullanımı açısından verimlidir.

```csharp
public class VeriOkuyucu
{
    private readonly string connectionString = "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;";
    
    public void MusterileriListele()
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            string sql = "SELECT CustomerID, CompanyName, ContactName FROM Customers";
            
            using (SqlCommand command = new SqlCommand(sql, connection))
            {
                try
                {
                    connection.Open();
                    
                    // DataReader oluştur
                    using (SqlDataReader reader = command.ExecuteReader())
                    {
                        // Sütun sayısını al
                        int fieldCount = reader.FieldCount;
                        
                        // Sütun başlıklarını yazdır
                        for (int i = 0; i < fieldCount; i++)
                        {
                            Console.Write($"{reader.GetName(i)}\t");
                        }
                        Console.WriteLine();
                        
                        // Verileri oku ve yazdır
                        while (reader.Read())
                        {
                            // Farklı veri okuma yöntemleri
                            string customerID = reader.GetString(0);
                            string companyName = reader.GetString(1);
                            string contactName = reader["ContactName"].ToString();
                            
                            Console.WriteLine($"{customerID}\t{companyName}\t{contactName}");
                        }
                    }
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Veri okuma hatası: {ex.Message}");
                }
            }
        }
    }
    
    public void CokluSonucKumeleriOku()
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            // Birden fazla SELECT sorgusu
            string sql = @"
                SELECT * FROM Customers WHERE Country = 'Germany';
                SELECT * FROM Orders WHERE OrderDate > '2020-01-01';
            ";
            
            using (SqlCommand command = new SqlCommand(sql, connection))
            {
                try
                {
                    connection.Open();
                    
                    using (SqlDataReader reader = command.ExecuteReader())
                    {
                        // İlk sonuç kümesini işle
                        Console.WriteLine("Alman Müşteriler:");
                        while (reader.Read())
                        {
                            Console.WriteLine($"{reader["CustomerID"]} - {reader["CompanyName"]}");
                        }
                        
                        // Sonraki sonuç kümesine geç
                        reader.NextResult();
                        
                        // İkinci sonuç kümesini işle
                        Console.WriteLine("\n2020 Sonrası Siparişler:");
                        while (reader.Read())
                        {
                            Console.WriteLine($"Sipariş No: {reader["OrderID"]}, Tarih: {reader["OrderDate"]}");
                        }
                    }
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Çoklu sonuç kümesi hatası: {ex.Message}");
                }
            }
        }
    }
}
```

### DataReader Performans İpuçları

```csharp
public class DataReaderPerformans
{
    private readonly string connectionString = "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;";
    
    public void PerformansliOkuma()
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            string sql = "SELECT * FROM LargeTable";
            
            using (SqlCommand command = new SqlCommand(sql, connection))
            {
                // Komut zaman aşımını ayarla
                command.CommandTimeout = 120; // 120 saniye
                
                try
                {
                    connection.Open();
                    
                    // SequentialAccess ile bellek kullanımını optimize et
                    using (SqlDataReader reader = command.ExecuteReader(CommandBehavior.SequentialAccess))
                    {
                        while (reader.Read())
                        {
                            // Sadece ihtiyaç duyulan sütunları oku
                            int id = reader.GetInt32(0);
                            string name = reader.GetString(1);
                            
                            // İşlem yap...
                        }
                    }
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Hata: {ex.Message}");
                }
            }
        }
    }
}
```

## DataSet ve DataAdapter

DataSet, veritabanı verilerinin bellekteki bir temsilini sağlar. DataAdapter ise, DataSet ile veritabanı arasındaki köprüdür.

### DataAdapter ve DataSet Kullanımı

```csharp
public class VeriAdaptoru
{
    private readonly string connectionString = "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;";
    
    public void DataSetKullanimi()
    {
        // DataSet oluştur
        DataSet dataSet = new DataSet("NorthwindData");
        
        // SqlDataAdapter oluştur
        using (SqlDataAdapter adapter = new SqlDataAdapter())
        {
            // Bağlantı ve komut nesnelerini ayarla
            using (SqlConnection connection = new SqlConnection(connectionString))
            {
                // Select komutu
                adapter.SelectCommand = new SqlCommand("SELECT * FROM Products", connection);
                
                // Insert komutu
                adapter.InsertCommand = new SqlCommand(
                    "INSERT INTO Products (ProductName, UnitPrice) VALUES (@ProductName, @UnitPrice)", 
                    connection);
                adapter.InsertCommand.Parameters.Add("@ProductName", SqlDbType.NVarChar, 40, "ProductName");
                adapter.InsertCommand.Parameters.Add("@UnitPrice", SqlDbType.Money, 0, "UnitPrice");
                
                try
                {
                    // DataSet'i doldur
                    adapter.Fill(dataSet, "Products");
                    
                    // DataTable'a erişim
                    DataTable productsTable = dataSet.Tables["Products"];
                    
                    // Verileri göster
                    foreach (DataRow row in productsTable.Rows)
                    {
                        Console.WriteLine($"Ürün: {row["ProductName"]}, Fiyat: {row["UnitPrice"]}");
                    }
                    
                    // Yeni satır ekle
                    DataRow newRow = productsTable.NewRow();
                    newRow["ProductName"] = "Yeni Ürün";
                    newRow["UnitPrice"] = 19.99;
                    productsTable.Rows.Add(newRow);
                    
                    // Değişiklikleri veritabanına kaydet
                    adapter.Update(dataSet, "Products");
                    
                    Console.WriteLine("Değişiklikler kaydedildi.");
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"DataSet hatası: {ex.Message}");
                }
            }
        }
    }
    
    public void CokluTabloCalisma()
    {
        DataSet northwindDS = new DataSet();
        
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            // Kategoriler için adapter
            SqlDataAdapter categoriesAdapter = new SqlDataAdapter(
                "SELECT * FROM Categories", connection);
            categoriesAdapter.Fill(northwindDS, "Categories");
            
            // Ürünler için adapter
            SqlDataAdapter productsAdapter = new SqlDataAdapter(
                "SELECT * FROM Products", connection);
            productsAdapter.Fill(northwindDS, "Products");
            
            // İlişki oluştur
            DataRelation relation = new DataRelation("CategoryProducts",
                northwindDS.Tables["Categories"].Columns["CategoryID"],
                northwindDS.Tables["Products"].Columns["CategoryID"]);
            
            northwindDS.Relations.Add(relation);
            
            // İlişkili verileri göster
            foreach (DataRow categoryRow in northwindDS.Tables["Categories"].Rows)
            {
                Console.WriteLine($"Kategori: {categoryRow["CategoryName"]}");
                
                // Bu kategoriye ait ürünleri bul
                DataRow[] productRows = categoryRow.GetChildRows(relation);
                
                foreach (DataRow productRow in productRows)
                {
                    Console.WriteLine($"  - {productRow["ProductName"]}");
                }
            }
        }
    }
}
```

## Parametreli Sorgular

Parametreli sorgular, SQL enjeksiyon saldırılarına karşı koruma sağlar ve performansı artırır.

```csharp
public class ParametreliSorgular
{
    private readonly string connectionString = "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;";
    
    public void ParametreliSorguCalistir()
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            // Parametreli SQL sorgusu
            string sql = "SELECT * FROM Products WHERE CategoryID = @CategoryID AND UnitPrice > @MinPrice";
            
            using (SqlCommand command = new SqlCommand(sql, connection))
            {
                // Parametreleri ekle
                command.Parameters.Add("@CategoryID", SqlDbType.Int).Value = 1;
                command.Parameters.Add("@MinPrice", SqlDbType.Money).Value = 20.0;
                
                try
                {
                    connection.Open();
                    
                    using (SqlDataReader reader = command.ExecuteReader())
                    {
                        while (reader.Read())
                        {
                            Console.WriteLine($"Ürün: {reader["ProductName"]}, Fiyat: {reader["UnitPrice"]}");
                        }
                    }
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Parametreli sorgu hatası: {ex.Message}");
                }
            }
        }
    }
    
    public void FarkliParametreTipleri()
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            string sql = @"
                INSERT INTO Orders (CustomerID, OrderDate, ShipName, ShipAddress)
                VALUES (@CustomerID, @OrderDate, @ShipName, @ShipAddress);
                SELECT SCOPE_IDENTITY();
            ";
            
            using (SqlCommand command = new SqlCommand(sql, connection))
            {
                // Farklı parametre tipleri
                command.Parameters.Add("@CustomerID", SqlDbType.NChar, 5).Value = "ALFKI";
                
                // DateTime parametresi
                command.Parameters.Add("@OrderDate", SqlDbType.DateTime).Value = DateTime.Now;
                
                // Metin parametresi
                SqlParameter shipNameParam = new SqlParameter("@ShipName", SqlDbType.NVarChar, 40);
                shipNameParam.Value = "Siparişi Alan Kişi";
                command.Parameters.Add(shipNameParam);
                
                // NULL değer içerebilen parametre
                SqlParameter addressParam = new SqlParameter("@ShipAddress", SqlDbType.NVarChar, 60);
                addressParam.Value = DBNull.Value; // NULL değer
                command.Parameters.Add(addressParam);
                
                try
                {
                    connection.Open();
                    
                    // Sorguyu çalıştır ve eklenen kaydın ID'sini al
                    object newOrderID = command.ExecuteScalar();
                    
                    Console.WriteLine($"Yeni sipariş ID: {newOrderID}");
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Parametre hatası: {ex.Message}");
                }
            }
        }
    }
}
```

## Transaction Yönetimi

Transaction'lar, birden fazla veritabanı işleminin atomik bir birim olarak çalışmasını sağlar.

```csharp
public class TransactionYonetimi
{
    private readonly string connectionString = "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;";
    
    public void TemelTransaction()
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            connection.Open();
            
            // Transaction başlat
            SqlTransaction transaction = connection.BeginTransaction();
            
            try
            {
                // İlk komut
                using (SqlCommand command1 = new SqlCommand(
                    "UPDATE Accounts SET Balance = Balance - 1000 WHERE AccountID = 1", 
                    connection, transaction))
                {
                    command1.ExecuteNonQuery();
                }
                
                // İkinci komut
                using (SqlCommand command2 = new SqlCommand(
                    "UPDATE Accounts SET Balance = Balance + 1000 WHERE AccountID = 2", 
                    connection, transaction))
                {
                    command2.ExecuteNonQuery();
                }
                
                // İşlemler başarılıysa commit et
                transaction.Commit();
                Console.WriteLine("Para transferi başarıyla tamamlandı.");
            }
            catch (Exception ex)
            {
                // Hata durumunda rollback yap
                transaction.Rollback();
                Console.WriteLine($"İşlem iptal edildi: {ex.Message}");
            }
        }
    }
    
    public void SavepointKullanimi()
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            connection.Open();
            
            SqlTransaction transaction = connection.BeginTransaction();
            
            try
            {
                // İlk işlem grubu
                using (SqlCommand command1 = new SqlCommand(
                    "INSERT INTO Logs (Message) VALUES ('İşlem başladı')", 
                    connection, transaction))
                {
                    command1.ExecuteNonQuery();
                }
                
                // Savepoint oluştur
                transaction.Save("Checkpoint1");
                
                try
                {
                    // Riskli işlem
                    using (SqlCommand command2 = new SqlCommand(
                        "UPDATE Products SET UnitPrice = UnitPrice * 1.2 WHERE CategoryID = 1", 
                        connection, transaction))
                    {
                        command2.ExecuteNonQuery();
                    }
                }
                catch (Exception)
                {
                    // Sadece riskli işlemi geri al
                    transaction.Rollback("Checkpoint1");
                    Console.WriteLine("Riskli işlem geri alındı, devam ediliyor...");
                }
                
                // Son işlem
                using (SqlCommand command3 = new SqlCommand(
                    "INSERT INTO Logs (Message) VALUES ('İşlem tamamlandı')", 
                    connection, transaction))
                {
                    command3.ExecuteNonQuery();
                }
                
                // Tüm işlemi commit et
                transaction.Commit();
                Console.WriteLine("İşlem başarıyla tamamlandı.");
            }
            catch (Exception ex)
            {
                // Tüm işlemi geri al
                transaction.Rollback();
                Console.WriteLine($"Tüm işlem iptal edildi: {ex.Message}");
            }
        }
    }
    
    public void TransactionIsolationLevel()
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            connection.Open();
            
            // Belirli bir izolasyon seviyesi ile transaction başlat
            SqlTransaction transaction = connection.BeginTransaction(IsolationLevel.ReadCommitted);
            
            try
            {
                using (SqlCommand command = new SqlCommand(
                    "SELECT * FROM Products WITH (UPDLOCK) WHERE ProductID = 1", 
                    connection, transaction))
                {
                    using (SqlDataReader reader = command.ExecuteReader())
                    {
                        if (reader.Read())
                        {
                            decimal currentPrice = (decimal)reader["UnitPrice"];
                            Console.WriteLine($"Mevcut fiyat: {currentPrice}");
                        }
                    }
                }
                
                using (SqlCommand updateCommand = new SqlCommand(
                    "UPDATE Products SET UnitPrice = UnitPrice * 1.1 WHERE ProductID = 1", 
                    connection, transaction))
                {
                    updateCommand.ExecuteNonQuery();
                }
                
                transaction.Commit();
                Console.WriteLine("Fiyat güncellendi.");
            }
            catch (Exception ex)
            {
                transaction.Rollback();
                Console.WriteLine($"İşlem iptal edildi: {ex.Message}");
            }
        }
    }
}
```

## ADO.NET ile Performans Optimizasyonu

ADO.NET kullanırken performansı artırmak için çeşitli teknikler kullanabilirsiniz.

```csharp
public class PerformansOptimizasyonu
{
    private readonly string connectionString = "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;";
    
    public void BulkInsert()
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            connection.Open();
            
            // SqlBulkCopy ile toplu veri ekleme
            using (SqlBulkCopy bulkCopy = new SqlBulkCopy(connection))
            {
                // Hedef tablo adını ayarla
                bulkCopy.DestinationTableName = "Products";
                
                // Sütun eşleştirmelerini ayarla
                bulkCopy.ColumnMappings.Add("ProductName", "ProductName");
                bulkCopy.ColumnMappings.Add("UnitPrice", "UnitPrice");
                bulkCopy.ColumnMappings.Add("CategoryID", "CategoryID");
                
                // Batch boyutunu ayarla
                bulkCopy.BatchSize = 1000;
                
                // Veri kaynağı olarak DataTable oluştur
                DataTable products = new DataTable();
                products.Columns.Add("ProductName", typeof(string));
                products.Columns.Add("UnitPrice", typeof(decimal));
                products.Columns.Add("CategoryID", typeof(int));
                
                // Örnek veri ekle
                for (int i = 0; i < 10000; i++)
                {
                    products.Rows.Add($"Ürün {i}", 10.99 + (i % 100), 1 + (i % 8));
                }
                
                try
                {
                    // Verileri toplu olarak ekle
                    bulkCopy.WriteToServer(products);
                    Console.WriteLine($"{products.Rows.Count} ürün başarıyla eklendi.");
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Toplu ekleme hatası: {ex.Message}");
                }
            }
        }
    }
    
    public void ConnectionPooling()
    {
        // Connection pooling varsayılan olarak etkindir
        // Bağlantı dizesinde kontrol edilebilir
        string poolingConnectionString = "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;Pooling=true;Min Pool Size=5;Max Pool Size=100;";
        
        // Çoklu bağlantı kullanımı
        for (int i = 0; i < 10; i++)
        {
            using (SqlConnection connection = new SqlConnection(poolingConnectionString))
            {
                try
                {
                    connection.Open();
                    // İşlemler...
                    Console.WriteLine($"Bağlantı {i+1} başarılı");
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"Bağlantı hatası: {ex.Message}");
                }
            } // Bağlantı havuza geri döner, gerçekten kapatılmaz
        }
    }
    
    public void AsenkronVeriErisimi()
    {
        // Asenkron veri erişimi örneği
        public async Task<List<Product>> GetProductsAsync()
        {
            List<Product> products = new List<Product>();
            
            using (SqlConnection connection = new SqlConnection(connectionString))
            {
                string sql = "SELECT ProductID, ProductName, UnitPrice FROM Products";
                
                using (SqlCommand command = new SqlCommand(sql, connection))
                {
                    // Asenkron bağlantı açma
                    await connection.OpenAsync();
                    
                    // Asenkron veri okuma
                    using (SqlDataReader reader = await command.ExecuteReaderAsync())
                    {
                        while (await reader.ReadAsync())
                        {
                            products.Add(new Product
                            {
                                ProductID = reader.GetInt32(0),
                                ProductName = reader.GetString(1),
                                UnitPrice = reader.GetDecimal(2)
                            });
                        }
                    }
                }
            }
            
            return products;
        }
    }
}
```

## Özet

ADO.NET, .NET platformunda veritabanı işlemleri için güçlü ve esnek bir altyapı sunar. Bu bölümde şunları öğrendik:

- **Connection ve Command Nesneleri**: Veritabanı bağlantısı ve SQL komutlarının yönetimi
- **DataReader Kullanımı**: Verimli, ileri yönlü veri okuma
- **DataSet ve DataAdapter**: Bağlantısız veri erişimi ve çoklu tablo işlemleri
- **Parametreli Sorgular**: Güvenli ve performanslı sorgu çalıştırma
- **Transaction Yönetimi**: Atomik veritabanı işlemleri
- **Performans Optimizasyonu**: Toplu işlemler ve asenkron veri erişimi

ADO.NET, ORM araçlarının sağladığı soyutlamaya ihtiyaç duymadığınız, düşük seviyeli veritabanı erişimi gerektiren durumlarda veya maksimum performans ihtiyacı olan senaryolarda hala değerli bir araçtır. 