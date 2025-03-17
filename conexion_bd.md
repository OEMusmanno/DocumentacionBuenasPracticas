# 🏆 **Buenas y Malas Prácticas en Conexión a Base de Datos en C#**  

Manejar correctamente la conexión a la base de datos es clave para **evitar fugas de memoria, bloqueos y problemas de rendimiento**. Veamos los errores comunes y cómo solucionarlos.  

---

## ❌ **Mala Práctica 1: No cerrar la conexión manualmente**  
### 🔴 **Problema:**  
- Dejar la conexión abierta puede generar **fugas de memoria y agotamiento de conexiones**.  
- Puede afectar el rendimiento si la base de datos tiene **un límite de conexiones concurrentes**.  

### 🚫 **Código Incorrecto:**  
```csharp
public void GetData()
{
    SqlConnection connection = new SqlConnection(_connectionString);
    connection.Open(); // ❌ Se abre pero nunca se cierra
    SqlCommand command = new SqlCommand("SELECT * FROM Users", connection);
    SqlDataReader reader = command.ExecuteReader();
}
```
🔴 **Problema:** Si se olvida cerrar la conexión, queda abierta hasta que el Garbage Collector la limpie, lo que **puede tardar** y generar cuellos de botella.

### ✅ **Solución:** Usar `using` para cerrar la conexión automáticamente  
```csharp
public void GetData()
{
    using (SqlConnection connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        using (SqlCommand command = new SqlCommand("SELECT * FROM Users", connection))
        {
            using (SqlDataReader reader = command.ExecuteReader())
            {
                while (reader.Read())
                {
                    Console.WriteLine(reader["Nombre"]);
                }
            }
        }
    } // ✅ La conexión se cierra automáticamente aquí
}
```
✔️ `using` garantiza que la conexión se cierre **siempre, incluso si hay una excepción**.  

---

## ❌ **Mala Práctica 2: No usar `async/await` en consultas a la base de datos**  
### 🔴 **Problema:**  
- Bloquea el hilo mientras se espera la respuesta de la base de datos.  
- Reduce la escalabilidad en aplicaciones web y de escritorio.  

### 🚫 **Código Incorrecto:**  
```csharp
public List<Usuario> GetUsuarios()
{
    List<Usuario> usuarios = new List<Usuario>();

    using (SqlConnection connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        using (SqlCommand command = new SqlCommand("SELECT * FROM Usuarios", connection))
        {
            using (SqlDataReader reader = command.ExecuteReader()) // ❌ Bloquea el hilo
            {
                while (reader.Read())
                {
                    usuarios.Add(new Usuario { Id = (int)reader["Id"], Nombre = (string)reader["Nombre"] });
                }
            }
        }
    }
    return usuarios;
}
```
🔴 **Problema:** `ExecuteReader()` es una operación I/O-bound, por lo que bloquear el hilo afecta el rendimiento.

### ✅ **Solución:** Usar `await` y métodos asincrónicos  
```csharp
public async Task<List<Usuario>> GetUsuariosAsync()
{
    List<Usuario> usuarios = new List<Usuario>();

    using (SqlConnection connection = new SqlConnection(_connectionString))
    {
        await connection.OpenAsync(); // ✅ No bloquea el hilo
        using (SqlCommand command = new SqlCommand("SELECT * FROM Usuarios", connection))
        {
            using (SqlDataReader reader = await command.ExecuteReaderAsync()) // ✅ Async
            {
                while (await reader.ReadAsync())
                {
                    usuarios.Add(new Usuario { Id = (int)reader["Id"], Nombre = (string)reader["Nombre"] });
                }
            }
        }
    }
    return usuarios;
}
```
✔️ **Libera el hilo mientras se espera la respuesta**, permitiendo que otras tareas sigan ejecutándose.  

---

## ❌ **Mala Práctica 3: No usar `Connection Pooling` correctamente**  
### 🔴 **Problema:**  
- Abrir y cerrar conexiones constantemente **aumenta la latencia**.  
- Puede **sobrecargar el servidor de base de datos** si se crean muchas conexiones nuevas.  

### 🚫 **Código Incorrecto:**  
```csharp
public async Task<List<Usuario>> GetUsuariosAsync()
{
    List<Usuario> usuarios = new List<Usuario>();

    for (int i = 0; i < 100; i++)
    {
        using (SqlConnection connection = new SqlConnection(_connectionString))
        {
            await connection.OpenAsync(); // ❌ Cada iteración crea una conexión nueva
            using (SqlCommand command = new SqlCommand("SELECT * FROM Usuarios", connection))
            {
                using (SqlDataReader reader = await command.ExecuteReaderAsync())
                {
                    while (await reader.ReadAsync())
                    {
                        usuarios.Add(new Usuario { Id = (int)reader["Id"], Nombre = (string)reader["Nombre"] });
                    }
                }
            }
        }
    }
    return usuarios;
}
```
🔴 **Problema:** **100 conexiones nuevas** en cada ejecución afectan el rendimiento y pueden superar el límite del `Connection Pool`.

### ✅ **Solución:** **Reutilizar conexiones con `Connection Pooling`**  
```csharp
public async Task<List<Usuario>> GetUsuariosAsync()
{
    List<Usuario> usuarios = new List<Usuario>();

    using (SqlConnection connection = new SqlConnection(_connectionString))
    {
        await connection.OpenAsync();
        for (int i = 0; i < 100; i++)
        {
            using (SqlCommand command = new SqlCommand("SELECT * FROM Usuarios", connection)) // ✅ Usa la misma conexión
            {
                using (SqlDataReader reader = await command.ExecuteReaderAsync())
                {
                    while (await reader.ReadAsync())
                    {
                        usuarios.Add(new Usuario { Id = (int)reader["Id"], Nombre = (string)reader["Nombre"] });
                    }
                }
            }
        }
    }
    return usuarios;
}
```
✔️ **Aprovecha el Connection Pool**, reduciendo la cantidad de conexiones abiertas.  

📌 **Importante:** .NET usa `Connection Pooling` por defecto en `SqlConnection`, por lo que **reutilizar conexiones dentro del mismo contexto es clave**.

---

## ❌ **Mala Práctica 4: Almacenar `SqlConnection` como variable de instancia**  
### 🔴 **Problema:**  
- Una conexión almacenada en un campo **se mantiene abierta por más tiempo del necesario**.  
- Puede **causar bloqueos y fugas de recursos**.  

### 🚫 **Código Incorrecto:**  
```csharp
public class UsuarioRepository
{
    private readonly SqlConnection _connection = new SqlConnection(_connectionString); // ❌ Mala práctica

    public async Task<List<Usuario>> GetUsuariosAsync()
    {
        await _connection.OpenAsync();
        using (SqlCommand command = new SqlCommand("SELECT * FROM Usuarios", _connection))
        {
            using (SqlDataReader reader = await command.ExecuteReaderAsync())
            {
                List<Usuario> usuarios = new List<Usuario>();
                while (await reader.ReadAsync())
                {
                    usuarios.Add(new Usuario { Id = (int)reader["Id"], Nombre = (string)reader["Nombre"] });
                }
                return usuarios;
            }
        }
    }
}
```
🔴 **Problema:** `_connection` puede quedar abierta por tiempo indefinido si no se maneja correctamente.

### ✅ **Solución:** Crear conexiones dentro de cada método  
```csharp
public async Task<List<Usuario>> GetUsuariosAsync()
{
    List<Usuario> usuarios = new List<Usuario>();

    using (SqlConnection connection = new SqlConnection(_connectionString))
    {
        await connection.OpenAsync();
        using (SqlCommand command = new SqlCommand("SELECT * FROM Usuarios", connection))
        {
            using (SqlDataReader reader = await command.ExecuteReaderAsync())
            {
                while (await reader.ReadAsync())
                {
                    usuarios.Add(new Usuario { Id = (int)reader["Id"], Nombre = (string)reader["Nombre"] });
                }
            }
        }
    }
    return usuarios;
}
```
✔️ **Cada método maneja su propia conexión**, evitando fugas de memoria.  

---

## 🎯 **Resumen de Buenas Prácticas**  
✅ **Siempre cerrar la conexión** con `using`.  
✅ **Usar `async/await`** para operaciones de base de datos.  
✅ **Reutilizar conexiones** en lugar de abrir nuevas constantemente.  
✅ **No almacenar `SqlConnection` como variable de instancia**.  

---
