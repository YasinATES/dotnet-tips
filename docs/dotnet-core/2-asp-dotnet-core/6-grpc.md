# gRPC

gRPC, Google tarafından geliştirilen, yüksek performanslı ve platform bağımsız bir uzak prosedür çağrısı (RPC) çerçevesidir. HTTP/2 üzerinde çalışır ve Protocol Buffers (protobuf) kullanarak veri serileştirme sağlar. Bu bölümde, .NET uygulamalarında gRPC kullanımını inceleyeceğiz.

## Protocol Buffers (protobuf)

Protocol Buffers, Google tarafından geliştirilen, yapılandırılmış verileri serileştirmek için kullanılan dil ve platform bağımsız bir mekanizmadır. JSON veya XML'e göre daha küçük, daha hızlı ve daha az belirsizdir.

### Protobuf Tanımı

Protobuf tanımları `.proto` uzantılı dosyalarda yapılır:

```protobuf
syntax = "proto3";

option csharp_namespace = "GrpcService";

package weather;

// Weather forecast service definition
service WeatherService {
  // Get weather forecast for a specific location
  rpc GetWeatherForecast (WeatherRequest) returns (WeatherResponse);
  
  // Get weather stream for a specific location
  rpc GetWeatherStream (WeatherRequest) returns (stream WeatherData);
}

// Request message with location information
message WeatherRequest {
  string location = 1;
  int32 days = 2;
}

// Response message containing weather forecasts
message WeatherResponse {
  repeated WeatherData forecasts = 1;
}

// Weather data for a specific day
message WeatherData {
  string date = 1;
  int32 temperature_c = 2;
  int32 temperature_f = 3;
  string summary = 4;
}
```

### C# Kod Üretimi

Protobuf tanımlarından C# kodu üretmek için, projenize Grpc.Tools paketini eklemeniz gerekir:

```xml
<ItemGroup>
  <Protobuf Include="Protos\weather.proto" GrpcServices="Server" />
  <PackageReference Include="Grpc.AspNetCore" Version="2.57.0" />
</ItemGroup>
```

Bu yapılandırma, derleme sırasında `.proto` dosyasından C# kodu üretir. Üretilen kodlar şunları içerir:
- Mesaj sınıfları (WeatherRequest, WeatherResponse, WeatherData)
- Servis temel sınıfları (WeatherService.WeatherServiceBase)
- İstemci sınıfları (WeatherService.WeatherServiceClient)

## Servis Tanımları

gRPC servisleri, `.proto` dosyasında tanımlanır ve sunucu tarafında uygulanır.

### Sunucu Tarafı Uygulama

```csharp
using Grpc.Core;
using GrpcService;

namespace GrpcService.Services
{
    public class WeatherService : WeatherService.WeatherServiceBase
    {
        private readonly ILogger<WeatherService> _logger;
        
        public WeatherService(ILogger<WeatherService> logger)
        {
            _logger = logger;
        }

        public override Task<WeatherResponse> GetWeatherForecast(WeatherRequest request, ServerCallContext context)
        {
            _logger.LogInformation("Getting weather forecast for {Location} for {Days} days", 
                request.Location, request.Days);
            
            // Create sample weather data
            var forecasts = Enumerable.Range(1, request.Days).Select(index => new WeatherData
            {
                Date = DateTime.UtcNow.AddDays(index).ToString("yyyy-MM-dd"),
                TemperatureC = Random.Shared.Next(-20, 55),
                Summary = GetRandomSummary()
            }).ToArray();
            
            // Set Fahrenheit temperature based on Celsius
            foreach (var forecast in forecasts)
            {
                forecast.TemperatureF = 32 + (int)(forecast.TemperatureC / 0.5556);
            }
            
            // Create and return the response
            var response = new WeatherResponse();
            response.Forecasts.AddRange(forecasts);
            
            return Task.FromResult(response);
        }
        
        private string GetRandomSummary()
        {
            var summaries = new[]
            {
                "Freezing", "Bracing", "Chilly", "Cool", "Mild", 
                "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
            };
            
            return summaries[Random.Shared.Next(summaries.Length)];
        }
    }
}
```

### Program.cs Yapılandırması

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add gRPC services to the container
builder.Services.AddGrpc();

var app = builder.Build();

// Configure the HTTP request pipeline
app.MapGrpcService<WeatherService>();
app.MapGet("/", () => "Communication with gRPC endpoints must be made through a gRPC client.");

app.Run();
```

### İstemci Tarafı Uygulama

```csharp
using Grpc.Net.Client;
using GrpcService;

// Create a channel
using var channel = GrpcChannel.ForAddress("https://localhost:5001");

// Create a client
var client = new WeatherService.WeatherServiceClient(channel);

// Make a request
var request = new WeatherRequest
{
    Location = "Istanbul",
    Days = 5
};

// Call the service
var response = await client.GetWeatherForecastAsync(request);

// Process the response
Console.WriteLine($"Weather forecast for {request.Location} for {request.Days} days:");

foreach (var forecast in response.Forecasts)
{
    Console.WriteLine($"Date: {forecast.Date}, Temp: {forecast.TemperatureC}°C ({forecast.TemperatureF}°F), Summary: {forecast.Summary}");
}
```

## Unary, Server Streaming, Client Streaming, Bi-directional Streaming

gRPC, dört farklı iletişim modelini destekler:

### Unary RPC

Unary RPC, istemcinin tek bir istek gönderdiği ve sunucunun tek bir yanıt döndürdüğü en basit RPC modelidir.

```protobuf
service CalculatorService {
  // Unary RPC
  rpc Add (AddRequest) returns (AddResponse);
}

message AddRequest {
  int32 a = 1;
  int32 b = 2;
}

message AddResponse {
  int32 result = 1;
}
```

Sunucu uygulaması:

```csharp
public override Task<AddResponse> Add(AddRequest request, ServerCallContext context)
{
    _logger.LogInformation("Adding {A} + {B}", request.A, request.B);
    
    var result = request.A + request.B;
    
    return Task.FromResult(new AddResponse { Result = result });
}
```

İstemci kullanımı:

```csharp
var response = await client.AddAsync(new AddRequest { A = 10, B = 20 });
Console.WriteLine($"Result: {response.Result}"); // Result: 30
```

### Server Streaming RPC

Server streaming RPC, istemcinin tek bir istek gönderdiği ve sunucunun bir dizi yanıt döndürdüğü modeldir.

```protobuf
service NumberService {
  // Server streaming RPC
  rpc GenerateNumbers (NumberRequest) returns (stream NumberResponse);
}

message NumberRequest {
  int32 max = 1;
}

message NumberResponse {
  int32 number = 1;
}
```

Sunucu uygulaması:

```csharp
public override async Task GenerateNumbers(NumberRequest request, 
    IServerStreamWriter<NumberResponse> responseStream, ServerCallContext context)
{
    _logger.LogInformation("Generating numbers up to {Max}", request.Max);
    
    // Generate numbers and stream them to the client
    for (int i = 1; i <= request.Max; i++)
    {
        // Check if the client has cancelled the request
        if (context.CancellationToken.IsCancellationRequested)
        {
            _logger.LogInformation("Request was cancelled");
            break;
        }
        
        await responseStream.WriteAsync(new NumberResponse { Number = i });
        
        // Simulate some work
        await Task.Delay(100);
    }
    
    _logger.LogInformation("Finished generating numbers");
}
```

İstemci kullanımı:

```csharp
using var call = client.GenerateNumbers(new NumberRequest { Max = 10 });

await foreach (var response in call.ResponseStream.ReadAllAsync())
{
    Console.WriteLine($"Received: {response.Number}");
}
```

### Client Streaming RPC

Client streaming RPC, istemcinin bir dizi istek gönderdiği ve sunucunun tek bir yanıt döndürdüğü modeldir.

```protobuf
service StatisticsService {
  // Client streaming RPC
  rpc CalculateAverage (stream MeasurementRequest) returns (AverageResponse);
}

message MeasurementRequest {
  double value = 1;
}

message AverageResponse {
  double average = 1;
  int32 count = 2;
}
```

Sunucu uygulaması:

```csharp
public override async Task<AverageResponse> CalculateAverage(
    IAsyncStreamReader<MeasurementRequest> requestStream, ServerCallContext context)
{
    double sum = 0;
    int count = 0;
    
    // Read all measurements from the client
    await foreach (var request in requestStream.ReadAllAsync())
    {
        _logger.LogInformation("Received measurement: {Value}", request.Value);
        
        sum += request.Value;
        count++;
    }
    
    double average = count > 0 ? sum / count : 0;
    
    _logger.LogInformation("Calculated average: {Average} from {Count} measurements", average, count);
    
    return new AverageResponse
    {
        Average = average,
        Count = count
    };
}
```

İstemci kullanımı:

```csharp
using var call = client.CalculateAverage();

// Send multiple measurements
for (int i = 1; i <= 5; i++)
{
    await call.RequestStream.WriteAsync(new MeasurementRequest { Value = i * 10 });
    await Task.Delay(100);
}

// Signal that we're done sending
await call.RequestStream.CompleteAsync();

// Get the response
var response = await call;
Console.WriteLine($"Average: {response.Average} (from {response.Count} measurements)");
```

### Bi-directional Streaming RPC

Bi-directional streaming RPC, hem istemcinin hem de sunucunun bir dizi mesaj gönderdiği modeldir.

```protobuf
service ChatService {
  // Bi-directional streaming RPC
  rpc Chat (stream ChatMessage) returns (stream ChatMessage);
}

message ChatMessage {
  string user = 1;
  string text = 2;
  string timestamp = 3;
}
```

Sunucu uygulaması:

```csharp
public override async Task Chat(
    IAsyncStreamReader<ChatMessage> requestStream,
    IServerStreamWriter<ChatMessage> responseStream,
    ServerCallContext context)
{
    // Create a queue for pending messages
    var pendingMessages = new ConcurrentQueue<ChatMessage>();
    
    // Start a background task to process and send messages
    var processingTask = Task.Run(async () =>
    {
        while (!context.CancellationToken.IsCancellationRequested)
        {
            if (pendingMessages.TryDequeue(out var message))
            {
                // Process the message (e.g., add server timestamp)
                message.Timestamp = DateTime.UtcNow.ToString("yyyy-MM-dd HH:mm:ss");
                
                // Send the message back to all clients
                await responseStream.WriteAsync(message);
                
                _logger.LogInformation("Sent message from {User}: {Text}", message.User, message.Text);
            }
            
            await Task.Delay(100);
        }
    });
    
    // Read incoming messages from this client
    await foreach (var message in requestStream.ReadAllAsync())
    {
        _logger.LogInformation("Received message from {User}: {Text}", message.User, message.Text);
        
        // Add the message to the queue for processing
        pendingMessages.Enqueue(message);
    }
    
    // Wait for the processing task to complete
    await processingTask;
}
```

İstemci kullanımı:

```csharp
using var call = client.Chat();

// Start a task to read responses
var responseTask = Task.Run(async () =>
{
    await foreach (var response in call.ResponseStream.ReadAllAsync())
    {
        Console.WriteLine($"[{response.Timestamp}] {response.User}: {response.Text}");
    }
});

// Send messages
string? input;
Console.Write("Enter your name: ");
string user = Console.ReadLine() ?? "Anonymous";

do
{
    Console.Write("Message (or 'exit' to quit): ");
    input = Console.ReadLine();
    
    if (!string.IsNullOrEmpty(input) && input != "exit")
    {
        await call.RequestStream.WriteAsync(new ChatMessage
        {
            User = user,
            Text = input
        });
    }
    
} while (input != "exit");

// Signal that we're done sending
await call.RequestStream.CompleteAsync();

// Wait for all responses
await responseTask;
```

## gRPC-Web

gRPC-Web, tarayıcılardan gRPC servislerine erişim sağlayan bir teknolojidir. Standart gRPC, HTTP/2 özelliklerini kullandığı için doğrudan tarayıcılardan kullanılamaz. gRPC-Web, bu sınırlamayı aşmak için bir proxy katmanı kullanır.

### Sunucu Tarafı Yapılandırma

Program.cs dosyasında gRPC-Web yapılandırması:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddGrpc();
builder.Services.AddGrpcWeb(options => options.GrpcWebEnabled = true);
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", builder =>
    {
        builder.AllowAnyOrigin()
               .AllowAnyMethod()
               .AllowAnyHeader();
    });
});

var app = builder.Build();

// Configure the HTTP request pipeline
app.UseCors("AllowAll");
app.UseGrpcWeb();
app.MapGrpcService<WeatherService>().EnableGrpcWeb();

app.Run();
```

### JavaScript İstemcisi

gRPC-Web için JavaScript istemcisi oluşturmak için, önce protobuf tanımlarından JavaScript kodu üretmeniz gerekir:

```bash
protoc -I=. weather.proto \
  --js_out=import_style=commonjs:./client \
  --grpc-web_out=import_style=commonjs,mode=grpcwebtext:./client
```

JavaScript ile gRPC-Web kullanımı:

```javascript
// Import the generated code
const {WeatherRequest} = require('./weather_pb.js');
const {WeatherServiceClient} = require('./weather_grpc_web_pb.js');

// Create a client
const client = new WeatherServiceClient('https://localhost:5001');

// Create a request
const request = new WeatherRequest();
request.setLocation('Istanbul');
request.setDays(5);

// Call the service
client.getWeatherForecast(request, {}, (err, response) => {
  if (err) {
    console.error('Error:', err);
    return;
  }
  
  // Process the response
  const forecastsList = response.getForecastsList();
  console.log(`Weather forecast for ${request.getLocation()} for ${request.getDays()} days:`);
  
  forecastsList.forEach(forecast => {
    console.log(`Date: ${forecast.getDate()}, Temp: ${forecast.getTemperatureC()}°C (${forecast.getTemperatureF()}°F), Summary: ${forecast.getSummary()}`);
  });
});
```

## Performance Özellikleri

gRPC, yüksek performanslı bir RPC çerçevesidir ve aşağıdaki özelliklere sahiptir:

### HTTP/2 Tabanlı

gRPC, HTTP/2 protokolü üzerinde çalışır ve şu avantajları sağlar:
- Çoklu istek ve yanıtları tek bir bağlantı üzerinden gönderebilme (multiplexing)
- Header sıkıştırma
- İkili veri aktarımı
- Sunucu push desteği

### Protobuf Serileştirme

Protocol Buffers, JSON veya XML'e göre daha verimli bir serileştirme formatıdır:
- Daha küçük mesaj boyutu (30-40% daha küçük)
- Daha hızlı serileştirme ve deserileştirme
- Şema tabanlı doğrulama

### Performans Optimizasyonları

gRPC uygulamalarında performansı artırmak için çeşitli optimizasyonlar yapılabilir:

#### Sunucu Tarafı Optimizasyonlar

```csharp
// Kestrel yapılandırması
builder.WebHost.ConfigureKestrel(options =>
{
    // HTTP/2 için maksimum akış sayısını artır
    options.Limits.Http2.MaxStreamsPerConnection = 100;
    
    // Maksimum istek gövdesi boyutunu ayarla
    options.Limits.MaxRequestBodySize = 4 * 1024 * 1024; // 4 MB
    
    // HTTP/2 PING çerçevelerini yapılandır
    options.Limits.Http2.KeepAlivePingDelay = TimeSpan.FromSeconds(30);
    options.Limits.Http2.KeepAlivePingTimeout = TimeSpan.FromSeconds(10);
});

// gRPC servis yapılandırması
builder.Services.AddGrpc(options =>
{
    // Sıkıştırma etkinleştir
    options.EnableDetailedErrors = true;
    options.MaxReceiveMessageSize = 4 * 1024 * 1024; // 4 MB
    options.MaxSendMessageSize = 4 * 1024 * 1024; // 4 MB
    options.CompressionProviders = new List<ICompressionProvider>
    {
        new GzipCompressionProvider(CompressionLevel.Optimal)
    };
    options.ResponseCompressionAlgorithm = "gzip";
    options.ResponseCompressionLevel = CompressionLevel.Optimal;
});
```

#### İstemci Tarafı Optimizasyonlar

```csharp
// Channel yapılandırması
var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
{
    // HTTP istemci yapılandırması
    HttpHandler = new SocketsHttpHandler
    {
        PooledConnectionIdleTimeout = TimeSpan.FromMinutes(5),
        KeepAlivePingDelay = TimeSpan.FromSeconds(60),
        KeepAlivePingTimeout = TimeSpan.FromSeconds(30),
        EnableMultipleHttp2Connections = true
    },
    
    // Sıkıştırma yapılandırması
    CompressionProviders = new List<ICompressionProvider>
    {
        new GzipCompressionProvider(CompressionLevel.Optimal)
    }
});

// İstemci yapılandırması
var client = new WeatherService.WeatherServiceClient(channel);
var callOptions = new CallOptions(deadline: DateTime.UtcNow.AddSeconds(30));

// İstek gönderme
var response = await client.GetWeatherForecastAsync(request, callOptions);
```

### Yük Testi Sonuçları

gRPC, REST API'lere göre genellikle daha iyi performans gösterir:

| Senaryo | gRPC | REST (JSON) | Fark |
|---------|------|-------------|------|
| Küçük Payload (50B) | 19,000 RPS | 13,500 RPS | gRPC 40% daha hızlı |
| Orta Payload (1KB) | 17,800 RPS | 11,200 RPS | gRPC 59% daha hızlı |
| Büyük Payload (10KB) | 15,000 RPS | 8,500 RPS | gRPC 76% daha hızlı |
| Streaming (1MB) | 450 MB/s | 250 MB/s | gRPC 80% daha hızlı |

Not: Bu sonuçlar örnek amaçlıdır ve gerçek performans, donanım, ağ ve uygulama yapılandırmasına göre değişebilir.

## Özet

gRPC, yüksek performanslı ve platform bağımsız bir RPC çerçevesidir. Protocol Buffers kullanarak verimli veri serileştirme sağlar ve HTTP/2 üzerinde çalışarak düşük gecikme süresi ve yüksek verimlilik sunar.

gRPC, dört farklı iletişim modelini destekler: unary, server streaming, client streaming ve bi-directional streaming. Bu modeller, farklı uygulama senaryolarına uygun çözümler sunar.

gRPC-Web, tarayıcılardan gRPC servislerine erişim sağlar ve modern web uygulamalarında kullanılabilir.

Performans açısından gRPC, REST API'lere göre genellikle daha iyi sonuçlar verir ve özellikle mikroservis mimarileri için ideal bir seçimdir. 