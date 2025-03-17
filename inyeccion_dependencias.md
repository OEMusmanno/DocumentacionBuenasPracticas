### 🏆 **Buenas y Malas Prácticas en Inyección de Dependencias en C#**  

La **Inyección de Dependencias (DI, Dependency Injection)** es un pilar del desarrollo en C#. Su uso correcto mejora la **mantenibilidad, testabilidad y escalabilidad** del código. Sin embargo, un mal uso puede generar **acoplamiento innecesario, problemas de rendimiento y código difícil de depurar**.  

---

## ❌ **Mala Práctica 1: Usar new dentro de los controladores o servicios**  
### 🔴 **Problema:**  
- Crear instancias con `new` dentro de un controlador o servicio genera **acoplamiento**.  
- Hace que la clase dependa directamente de la implementación en lugar de una abstracción.  
- Dificulta las pruebas unitarias, ya que no se puede inyectar un mock fácilmente.  

### 🚫 **Código Incorrecto:**  
```csharp
public class UsuarioController : ControllerBase
{
    private readonly UsuarioService _usuarioService;

    public UsuarioController()
    {
        _usuarioService = new UsuarioService(); // ❌ Instanciación directa
    }

    public IActionResult ObtenerUsuarios()
    {
        var usuarios = _usuarioService.ObtenerTodos();
        return Ok(usuarios);
    }
}
```

### ✅ **Solución:** Usar Inyección de Dependencias  
```csharp
public class UsuarioController : ControllerBase
{
    private readonly IUsuarioService _usuarioService;

    public UsuarioController(IUsuarioService usuarioService) // ✅ Inyección por constructor
    {
        _usuarioService = usuarioService;
    }

    public IActionResult ObtenerUsuarios()
    {
        var usuarios = _usuarioService.ObtenerTodos();
        return Ok(usuarios);
    }
}
```
✔️ **Beneficios:**  
- **Se desacopla la implementación** y se puede cambiar `UsuarioService` sin modificar el controlador.  
- **Facilita las pruebas unitarias** inyectando un mock de `IUsuarioService`.  

---

## ❌ **Mala Práctica 2: Usar dependencias estáticas**  
### 🔴 **Problema:**  
- **Las clases estáticas no pueden ser reemplazadas fácilmente**.  
- No se pueden **inyectar dependencias** en una clase estática.  

### 🚫 **Código Incorrecto:**  
```csharp
public class UsuarioController : ControllerBase
{
    public IActionResult ObtenerUsuarios()
    {
        var usuarios = UsuarioService.ObtenerTodos(); // ❌ Clase estática
        return Ok(usuarios);
    }
}
```

### ✅ **Solución:** Inyectar la dependencia correctamente  
```csharp
public class UsuarioController : ControllerBase
{
    private readonly IUsuarioService _usuarioService;

    public UsuarioController(IUsuarioService usuarioService)
    {
        _usuarioService = usuarioService;
    }

    public IActionResult ObtenerUsuarios()
    {
        var usuarios = _usuarioService.ObtenerTodos(); // ✅ Servicio inyectado
        return Ok(usuarios);
    }
}
```
✔️ **Beneficios:**  
- Evita el uso de **clases estáticas acopladas**.  
- **Permite cambiar la implementación** sin modificar el controlador.  

---

## ❌ **Mala Práctica 3: No registrar las dependencias en el contenedor**  
### 🔴 **Problema:**  
- Si una dependencia **no está registrada en el contenedor**, se produce un error en tiempo de ejecución.  
- Puede ser confuso para otros desarrolladores.  

### 🚫 **Código Incorrecto:**  
```csharp
services.AddTransient<UsuarioService>(); // ❌ Falta la interfaz
```

### ✅ **Solución:** Registrar correctamente las dependencias  
```csharp
services.AddTransient<IUsuarioService, UsuarioService>(); // ✅ Correcto
```
✔️ **Reglas:**  
- **AddTransient** → Nueva instancia en cada solicitud.  
- **AddScoped** → Misma instancia dentro de una solicitud HTTP.  
- **AddSingleton** → Misma instancia durante toda la vida de la aplicación.  

---

## ❌ **Mala Práctica 4: Usar AddSingleton para dependencias con estado**  
### 🔴 **Problema:**  
- Si una dependencia maneja **datos de usuario o conexiones**, usar `AddSingleton` puede causar problemas de concurrencia.  

### 🚫 **Código Incorrecto:**  
```csharp
services.AddSingleton<IUsuarioService, UsuarioService>(); // ❌ Podría causar problemas en entornos multiusuario
```

### ✅ **Solución:** Usar AddScoped o AddTransient  
```csharp
services.AddScoped<IUsuarioService, UsuarioService>(); // ✅ Para entidades con estado
```
✔️ **Reglas:**  
- **Singleton** solo para servicios sin estado (ej: configuración, caché).  
- **Scoped o Transient** para servicios con datos específicos de una solicitud.  

---

## ❌ **Mala Práctica 5: Inyectar demasiadas dependencias en un constructor**  
### 🔴 **Problema:**  
- Si un constructor recibe **demasiadas dependencias**, la clase **está haciendo demasiado**.  
- Es una señal de que **debería dividirse en múltiples responsabilidades**.  

### 🚫 **Código Incorrecto:**  
```csharp
public class PedidoService
{
    public PedidoService(IUsuarioService usuarioService, IEmailService emailService, ILogService logService, IPagoService pagoService)
    {
        // ❌ Demasiadas dependencias
    }
}
```

### ✅ **Solución:** Aplicar el patrón **Facade o Mediator**  
```csharp
public class PedidoFacade
{
    private readonly IUsuarioService _usuarioService;
    private readonly IPagoService _pagoService;

    public PedidoFacade(IUsuarioService usuarioService, IPagoService pagoService)
    {
        _usuarioService = usuarioService;
        _pagoService = pagoService;
    }

    public void ProcesarPedido()
    {
        _usuarioService.Validar();
        _pagoService.Procesar();
    }
}
```
✔️ **Beneficio:**  
- Reduce la cantidad de dependencias inyectadas **directamente en una clase**.  

---

## 🎯 **Resumen de Buenas Prácticas en Inyección de Dependencias**  
✅ **Evitar `new` dentro de controladores y servicios**, inyectar en el constructor.  
✅ **No usar clases estáticas**, preferir inyección de dependencias.  
✅ **Registrar todas las dependencias en `Program.cs`** (`AddTransient`, `AddScoped`, `AddSingleton`).  
✅ **No usar `Singleton` para dependencias con estado**.  
✅ **Evitar inyectar demasiadas dependencias en una clase**, usar **Facade o Mediator**.  

---
