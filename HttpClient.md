Se realizo una comparación detallada de `HttpClient` y `HttpClientFactory`, incluyendo cómo manejan las conexiones, los valores por defecto en .NET y sus implicancias en el rendimiento.

---

## 🚀 **Comparación entre `HttpClient` y `HttpClientFactory`**

| Característica | `HttpClient` (uso directo) | `HttpClientFactory` (uso recomendado) |
|--------------|-------------------------|-------------------------------|
| **Gestión de conexiones** | Crea su propio pool de conexiones | Reutiliza `HttpMessageHandler` para optimizar conexiones |
| **Reutilización** | Si se crea una nueva instancia por cada request, puede generar agotamiento de sockets | Gestiona instancias y conexiones automáticamente |
| **Problema de agotamiento de sockets** | Ocurre si se crean muchas instancias sin reutilizarlas | Evita este problema al manejar bien los handlers |
| **Tiempo de vida de los `HttpMessageHandler`** | Depende del tiempo de vida del `HttpClient` | 2 minutos por defecto (`HttpClientFactoryOptions.HandlerLifetime`) |
| **Límite de conexiones simultáneas** | Depende de `ServicePointManager.DefaultConnectionLimit` (Framework) o `SocketsHttpHandler.MaxConnectionsPerServer` (.NET Core y .NET 5+) | Usa `SocketsHttpHandler`, con `MaxConnectionsPerServer = int.MaxValue` por defecto |
| **Configuración y personalización** | Manual y por instancia | Centralizada, soporta Polly para reintentos y circuit breakers |
| **Uso en aplicaciones de larga duración** | Puede ser problemático si se crean y descartan instancias frecuentemente | Se recomienda para apps de alto tráfico |
| **Uso recomendado** | Solo si la instancia es reutilizada a lo largo de toda la app | Siempre que sea posible, especialmente en .NET Core y .NET 5+ |
| Configuración | `HttpClient` (sin factory) | `HttpClientFactory` (con `SocketsHttpHandler`) |
| **Timeout por defecto** | `100 segundos` (`Timeout = TimeSpan.FromSeconds(100)`) | Igual (`Timeout = TimeSpan.FromSeconds(100)`) |
| **Límite de conexiones por host** | En .NET Framework, depende de `ServicePointManager.DefaultConnectionLimit` (2 por host por defecto). En .NET Core y .NET 5+, es **ilimitado** | `SocketsHttpHandler.MaxConnectionsPerServer = int.MaxValue` (∞ conexiones por host) |
| **Pool de conexiones** | Cada instancia tiene su propio pool (no compartido) | Pool compartido mediante `HttpMessageHandler` reutilizable |
| **Tiempo de vida del handler** | Hasta que se elimine el `HttpClient` | `HandlerLifetime = 2 minutos` (después de esto, se crea un nuevo handler) |
| **Keep-Alive** | Habilitado por defecto | También habilitado (`SocketsHttpHandler.PooledConnectionLifetime = Timeout.InfiniteTimeSpan`) |
| **Reuse de conexiones** | Solo si se reutiliza la misma instancia | Manejado automáticamente, evita agotamiento de sockets |
---

## 📌 **Detalles de comportamiento en .NET**
### 1️⃣ **`HttpClient` sin `HttpClientFactory`**
- No tiene límite de conexiones en .NET Core y .NET 5+ (antes dependía de `ServicePointManager.DefaultConnectionLimit`).
- Cada instancia de `HttpClient` tiene su propio `HttpMessageHandler` y su propio pool de conexiones.
- Si no se reutiliza una misma instancia, se crean nuevas conexiones constantemente, lo que puede llevar a **Socket Exhaustion**.
- No se recomienda usar `using` para `HttpClient`, ya que al ser eliminado **no cierra inmediatamente las conexiones abiertas**.

### 2️⃣ **`HttpClientFactory` (`IHttpClientFactory`)**
- Usa `SocketsHttpHandler` internamente (más eficiente que `HttpClientHandler` de .NET Framework).
- **Handler reutilizable**: Las conexiones son gestionadas en un pool y pueden ser reutilizadas por diferentes instancias de `HttpClient`.
- **Tiempo de vida del handler:** Por defecto, cada `HttpMessageHandler` dura **2 minutos** (`HandlerLifetime = TimeSpan.FromMinutes(2)`). Esto permite evitar problemas de DNS cacheado por mucho tiempo.

---

## 🔥 **Ejemplo de mal uso de `HttpClient`**
Esto genera un agotamiento de sockets si se ejecuta muchas veces:

```csharp
for (int i = 0; i < 100; i++)
{
    using var client = new HttpClient();
    var response = await client.GetAsync("https://api.github.com");
    Console.WriteLine(response.StatusCode);
}
```
Cada `HttpClient` crea nuevas conexiones y deja abiertas las anteriores hasta que el GC las libere.

---

## ✅ **Uso correcto con `HttpClientFactory`**
Esta es la mejor práctica en .NET moderno:

```csharp
  public Test(ILogger<TestController> logger, IHttpClientFactory httpClientFactory)
  {
      _logger = logger;
      _httpClientFactory = httpClientFactory;
  }

 [HttpGet("httpclientfactory")]
    public async Task<IActionResult> GetWithHttpClientFactory()
    {
        var httpClient = _httpClientFactory.CreateClient("ApiExample");

        try
        {
            var response = await httpClient.GetStringAsync("https://api.example.com/data");
            return Ok(response);
        }
        catch (HttpRequestException ex)
        {
            _logger.LogError(ex, "Error en la solicitud HTTP");
            return StatusCode(500, "Error al obtener los datos.");
        }
    }

```
Con `HttpClientFactory`, todas las instancias creadas con `CreateClient()` reutilizan conexiones de manera eficiente.

---

## 🔍 **Valores por defecto en .NET** (Timeouts, conexiones, pool, etc.)

| Configuración | `HttpClient` (sin factory) | `HttpClientFactory` (con `SocketsHttpHandler`) |
|--------------|-------------------------|-------------------------------|
| **Timeout por defecto** | `100 segundos` (`Timeout = TimeSpan.FromSeconds(100)`) | Igual (`Timeout = TimeSpan.FromSeconds(100)`) |
| **Límite de conexiones por host** | En .NET Framework, depende de `ServicePointManager.DefaultConnectionLimit` (2 por host por defecto). En .NET Core y .NET 5+, es **ilimitado** | `SocketsHttpHandler.MaxConnectionsPerServer = int.MaxValue` (∞ conexiones por host) |
| **Pool de conexiones** | Cada instancia tiene su propio pool (no compartido) | Pool compartido mediante `HttpMessageHandler` reutilizable |
| **Tiempo de vida del handler** | Hasta que se elimine el `HttpClient` | `HandlerLifetime = 2 minutos` (después de esto, se crea un nuevo handler) |
| **Keep-Alive** | Habilitado por defecto | También habilitado (`SocketsHttpHandler.PooledConnectionLifetime = Timeout.InfiniteTimeSpan`) |
| **Reuse de conexiones** | Solo si se reutiliza la misma instancia | Manejado automáticamente, evita agotamiento de sockets |

---

### ⏳ **Timeouts**
- `HttpClient.Timeout` (100 segundos por defecto) define cuánto tiempo se espera antes de cancelar la petición.
- Se puede configurar por petición:
  
  ```csharp
  var client = new HttpClient { Timeout = TimeSpan.FromSeconds(30) };
  ```

- En `HttpClientFactory`, el timeout es configurable por cliente:

  ```csharp
  builder.Services.AddHttpClient("MiCliente", client =>
  {
      client.Timeout = TimeSpan.FromSeconds(30);
  });
  ```

---

### 🌍 **Límite de conexiones y Pool de Conexiones**
- En `.NET Framework`, el límite de conexiones es **2 por host** (por `ServicePointManager.DefaultConnectionLimit`).
- En `.NET Core 2.1+`, `SocketsHttpHandler.MaxConnectionsPerServer = int.MaxValue`, lo que significa **conexiones ilimitadas por host**.
- `HttpClientFactory` **reutiliza conexiones** en lugar de crear nuevas.

Si querés limitar las conexiones por host en `HttpClientFactory`:

```csharp
builder.Services.AddHttpClient("MiCliente")
    .ConfigurePrimaryHttpMessageHandler(() => new SocketsHttpHandler
    {
        MaxConnectionsPerServer = 10 // Limita a 10 conexiones por host
    });
```

---

### 🔄 **Keep-Alive y Reutilización de Conexiones**
- `Keep-Alive` está activado por defecto en `HttpClient` y `HttpClientFactory`.
- `HttpClientFactory` reutiliza conexiones y permite definir cuánto tiempo deben vivir:

  ```csharp
  builder.Services.AddHttpClient("MiCliente")
      .ConfigurePrimaryHttpMessageHandler(() => new SocketsHttpHandler
      {
          PooledConnectionLifetime = TimeSpan.FromMinutes(5) // Cierra conexiones cada 5 min
      });
  ```

Esto es útil para evitar problemas de **DNS cacheado** (si un servicio cambia de IP, las conexiones antiguas pueden seguir apuntando a la IP vieja).


---

## 🔹 **Configuración en `HttpClientFactory` (`SocketsHttpHandler`)**  

| **Parámetro (C#)** | **Descripción** | **Propiedad en .NET** | **Valor por Defecto** |
|--------------------|----------------|----------------------|----------------------|
| **Timeout** | Tiempo máximo de espera antes de cancelar la solicitud. | `HttpClient.Timeout` | `100s` |
| **MaxConnectionsPerServer** | Máximo de conexiones simultáneas por servidor. | `SocketsHttpHandler.MaxConnectionsPerServer` | `int.MaxValue` (Ilimitado) |
| **PooledConnectionLifetime** | Tiempo máximo antes de reciclar una conexión en el pool (útil para DNS refresh). | `SocketsHttpHandler.PooledConnectionLifetime` | `Timeout.InfiniteTimeSpan` (No se reciclan) |
| **PooledConnectionIdleTimeout** | Tiempo máximo que una conexión inactiva puede permanecer en el pool antes de cerrarse. | `SocketsHttpHandler.PooledConnectionIdleTimeout` | `2 minutos` |
| **HandlerLifetime** | Tiempo de vida del `HttpMessageHandler`, después del cual se crea uno nuevo. | `IHttpClientFactory.HandlerLifetime` | `2 minutos` |
| **AllowAutoRedirect** | Indica si se siguen automáticamente las redirecciones HTTP. | `SocketsHttpHandler.AllowAutoRedirect` | `true` |
| **MaxAutomaticRedirections** | Máximo de redirecciones automáticas permitidas. | `SocketsHttpHandler.MaxAutomaticRedirections` | `50` |
| **AutomaticDecompression** | Habilita la descompresión automática de respuestas (`gzip`, `deflate`). | `SocketsHttpHandler.AutomaticDecompression` | `None` |
| **UseCookies** | Habilita el manejo automático de cookies. | `SocketsHttpHandler.UseCookies` | `true` |
| **CookieContainer** | Contenedor de cookies para almacenar y enviar automáticamente en las solicitudes. | `SocketsHttpHandler.CookieContainer` | Instancia vacía |
| **UseProxy** | Habilita el uso de un proxy configurado. | `SocketsHttpHandler.UseProxy` | `true` |
| **Proxy** | Proxy personalizado para las solicitudes. | `SocketsHttpHandler.Proxy` | `null` (sin proxy) |
| **DefaultProxyCredentials** | Credenciales predeterminadas para autenticación en el proxy. | `SocketsHttpHandler.DefaultProxyCredentials` | `null` |
| **Expect100Continue** | Indica si se usa `Expect: 100-continue` en peticiones POST grandes. | `SocketsHttpHandler.Expect100Continue` | `true` |


---

## 🎯 **Conclusión**
Si bien `HttpClient` puede usarse correctamente si se reutiliza bien la instancia, `HttpClientFactory` es la mejor opción en .NET moderno porque **maneja el pool de conexiones y evita problemas de rendimiento y escalabilidad**. Además, permite **configuración centralizada, inyección de dependencias y resiliencia con Polly**.

- **Timeout:** `100s` por defecto, configurable en `HttpClientFactory`.
- **Límite de conexiones por host:** `∞` en `.NET Core+`, pero configurable (`MaxConnectionsPerServer`).
- **Pool de conexiones:** `HttpClientFactory` reutiliza conexiones, mientras que `HttpClient` crea una nueva si no se reutiliza la instancia.
- **Keep-Alive:** Activado por defecto; configurable (`PooledConnectionLifetime`).

