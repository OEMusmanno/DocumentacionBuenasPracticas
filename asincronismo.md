# 🏆 **Buenas y Malas Prácticas en Asincronismo (`async/await`)**

El asincronismo en C# es clave para mejorar la **eficiencia y rendimiento** de las aplicaciones. Sin embargo, un mal uso puede generar **bloqueos, deadlocks y problemas de escalabilidad**.

---

## ❌ **Mala Práctica 1: Bloquear el hilo con `.Result` o `.Wait()`**  
### 🔴 **Problema:**  
- **Causa bloqueos** en el hilo principal.  
- Puede generar **deadlocks** en aplicaciones ASP.NET.  

### 🚫 **Código Incorrecto:**  
```csharp
public string GetData()
{
    return _httpClient.GetStringAsync("https://api.example.com/data").Result; 
    // ❌ Bloquea el hilo
}
```

### ✅ **Solución:** Usar `await` correctamente  
```csharp
public async Task<string> GetDataAsync()
{
    return await _httpClient.GetStringAsync("https://api.example.com/data"); 
    // ✅ No bloquea el hilo
}
```
✔️ Permite que el hilo principal siga ejecutando otras tareas sin quedar bloqueado.

---

## ❌ **Mala Práctica 2: No encadenar correctamente las llamadas `async`**  
### 🔴 **Problema:**  
- Si se ejecutan varios `await` en secuencia, cada tarea espera a que la anterior termine.  
- **Ineficiente** si las tareas no dependen entre sí.  

### 🚫 **Código Incorrecto:**  
```csharp
public async Task ProcesarDatosAsync()
{
    var datos1 = await ObtenerDatos1Async(); // 🕒 Espera que termine
    var datos2 = await ObtenerDatos2Async(); // 🕒 Espera que termine
    var datos3 = await ObtenerDatos3Async(); // 🕒 Espera que termine
}
```
🔴 **Tiempo total = Tarea1 + Tarea2 + Tarea3**

### ✅ **Solución:** Usar `Task.WhenAll` para paralelismo  
```csharp
public async Task ProcesarDatosEnParaleloAsync()
{
    var tarea1 = ObtenerDatos1Async();
    var tarea2 = ObtenerDatos2Async();
    var tarea3 = ObtenerDatos3Async();

    var resultados = await Task.WhenAll(tarea1, tarea2, tarea3); // ✅ Se ejecutan en paralelo
}
```
✔️ **Ejecuta las tareas en simultáneo**, reduciendo el tiempo total de ejecución.

---

## ❌ **Mala Práctica 3: No manejar excepciones en métodos `async`**  
### 🔴 **Problema:**  
- Una excepción sin capturar en un método `async` puede **crashear la aplicación**.  

### 🚫 **Código Incorrecto:**  
```csharp
public async Task<string> GetDataAsync()
{
    return await _httpClient.GetStringAsync("https://api.example.com/data"); 
    // ❌ No maneja errores
}
```

### ✅ **Solución:** Usar `try-catch` con `await`  
```csharp
public async Task<string> GetDataAsync()
{
    try
    {
        return await _httpClient.GetStringAsync("https://api.example.com/data");
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"Error en la solicitud: {ex.Message}");
        return string.Empty;
    }
}
```
✔️ **Maneja errores** sin detener la ejecución del programa.

---

## ❌ **Mala Práctica 4: Usar `async void` en métodos públicos**  
### 🔴 **Problema:**  
- `async void` **no devuelve un `Task`**, lo que impide capturar excepciones y errores.  
- Puede **romper el flujo de ejecución** en ASP.NET y otros entornos.  

### 🚫 **Código Incorrecto:**  
```csharp
public async void CargarDatos()
{
    await Task.Delay(1000);
    Console.WriteLine("Datos cargados");
}
```
🔴 **Problema:** No se puede capturar excepciones ni manejar el estado de la tarea.

### ✅ **Solución:** Usar `async Task`  
```csharp
public async Task CargarDatosAsync()
{
    await Task.Delay(1000);
    Console.WriteLine("Datos cargados");
}
```
✔️ Permite **esperar correctamente** la ejecución del método.

---

## ❌ **Mala Práctica 5: Usar `Task.Run` innecesariamente**  
### 🔴 **Problema:**  

En C#, `Task.Run` se usa para ejecutar **tareas intensivas en CPU** en un hilo del **ThreadPool**. Sin embargo, cuando se usa con operaciones de **entrada/salida (I/O-bound)**, como solicitudes HTTP o consultas a bases de datos, no aporta ningún beneficio y puede **empeorar el rendimiento**.

### 📌 **Concepto clave: Diferencia entre CPU-bound e I/O-bound**  
- **CPU-bound**: Tareas que requieren **cómputo intensivo**, como procesamiento de imágenes o cálculos matemáticos pesados.  
- **I/O-bound**: Tareas que esperan una respuesta de una fuente externa, como una API o una base de datos.

Las **operaciones I/O-bound** ya son asincrónicas por naturaleza, por lo que `await` libera el hilo sin necesidad de usar `Task.Run`.

---

### 🚫 **Ejemplo incorrecto: Usar `Task.Run` en una solicitud HTTP**  
```csharp
public async Task<string> GetDataAsync()
{
    return await Task.Run(() => _httpClient.GetStringAsync("https://api.example.com/data")); // ❌ Mal uso de Task.Run
}
```

### 🔴 **¿Por qué esto es un problema?**  
1. **Crea un hilo extra en el `ThreadPool`** para ejecutar código que ya es asincrónico.  
2. **No mejora el rendimiento**, porque el método `GetStringAsync` ya es asincrónico y no bloquea el hilo principal.  
3. **Aumenta el consumo de CPU y memoria**, al agregar una capa innecesaria de administración de hilos.  

En una aplicación con muchas solicitudes concurrentes, esto puede sobrecargar el `ThreadPool` y hacer que las tareas realmente CPU-bound tengan menos recursos.

---

### ✅ **Solución correcta: Usar `await` directamente**
```csharp
public async Task<string> GetDataAsync()
{
    return await _httpClient.GetStringAsync("https://api.example.com/data"); // ✅ No usa Task.Run innecesariamente
}
```

### **🔹 Explicación:**  
✔️ `await` **libera el hilo** mientras la solicitud HTTP se procesa en segundo plano.  
✔️ No se crea un hilo adicional, **optimiza el uso de recursos**.  
✔️ El código sigue siendo completamente asincrónico y escalable.

---

### ⚠️ **¿Cuándo SÍ usar `Task.Run`?**  
El uso de `Task.Run` **está justificado** en tareas que requieren **mucho procesamiento en la CPU**, como:  
✔️ Procesamiento de imágenes.  
✔️ Cifrado o compresión de datos.  
✔️ Algoritmos pesados de IA o Machine Learning.  

Ejemplo válido de `Task.Run`:  
```csharp
public async Task ProcesarImagenAsync(byte[] imagen)
{
    await Task.Run(() => FiltroImagen(imagen)); // ✅ Aquí SÍ tiene sentido usar Task.Run
}
```
---

## 🎯 **Resumen de Buenas Prácticas**  
✅ Usar `await` en toda la cadena de llamadas para evitar bloqueos.  
✅ Evitar `.Result`, `.Wait()` y `Task.Run()` innecesario.  
❌ No uses `Task.Run` en operaciones **I/O-bound** como HTTP o bases de datos.  
✅ Ejecutar tareas en paralelo con `Task.WhenAll` cuando sea posible.  
✅ Manejar excepciones con `try-catch`.  
✅ Evitar `async void`, usar siempre `async Task`.  

---
