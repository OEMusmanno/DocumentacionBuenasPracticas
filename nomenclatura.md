# 🏆 **Buenas y Malas Prácticas en Nomenclatura en C#**  

La nomenclatura en C# es clave para la **legibilidad, mantenibilidad y coherencia del código**. Aplicar las convenciones correctas **facilita el trabajo en equipo y evita errores**.  

---

## ❌ **Mala Práctica 1: No seguir el PascalCase y camelCase**  
### 🔴 **Problema:**  
- C# tiene convenciones de nomenclatura estándar. No seguirlas **dificulta la lectura**.  
- Los nombres inconsistentes hacen que el código **se vea desordenado y confuso**.  

### 🚫 **Código Incorrecto:**  
```csharp
public class usuarioRepositorio // ❌ Clases en minúsculas
{
    public string nombre_usuario; // ❌ Campos con guiones bajos y sin camelCase

    public void obtenerUsuarios() // ❌ Métodos en minúsculas
    {
        Console.WriteLine("Obteniendo usuarios...");
    }
}
```
🔴 **Problema:**  
- `usuarioRepositorio` debería estar en **PascalCase** (`UsuarioRepositorio`).  
- `nombre_usuario` debería usar **camelCase** (`nombreUsuario`).  
- `obtenerUsuarios()` debería empezar con **mayúscula** (`ObtenerUsuarios`).  

### ✅ **Solución:** Aplicar PascalCase y camelCase  
```csharp
public class UsuarioRepositorio // ✅ Clases en PascalCase
{
    public string NombreUsuario; // ✅ Campos en PascalCase (para propiedades públicas)

    public void ObtenerUsuarios() // ✅ Métodos en PascalCase
    {
        Console.WriteLine("Obteniendo usuarios...");
    }
}
```
✔️ **Reglas básicas:**  
- **Clases, interfaces, métodos y propiedades** → PascalCase (`MiClase`, `ObtenerUsuarios`).  
- **Variables y parámetros** → camelCase (`nombreUsuario`, `idUsuario`).  

---

## ❌ **Mala Práctica 2: Usar nombres poco descriptivos**  
### 🔴 **Problema:**  
- Usar nombres cortos como `x`, `y`, `a1` **no explica qué hace una variable**.  
- Hace que el código **sea difícil de entender sin contexto**.  

### 🚫 **Código Incorrecto:**  
```csharp
int x = 100; // ❌ Nombre sin contexto
string str = "Hola"; // ❌ Nombre genérico
var p = ObtenerUsuario(); // ❌ ¿Qué es "p"?  
```

### ✅ **Solución:** Usar nombres descriptivos  
```csharp
int tiempoDeEsperaMs = 100; // ✅ Describe su uso
string mensajeBienvenida = "Hola"; // ✅ Es claro
var usuario = ObtenerUsuario(); // ✅ Ahora es entendible
```
✔️ **Reglas básicas:**  
- **Usar nombres que describan el propósito** (`usuario`, `cantidadClientes`, `totalVentas`).  
- **Evitar abreviaciones innecesarias** (`cantidadVentas`, no `cntVentas`).  

---

## ❌ **Mala Práctica 3: No usar prefijos correctos en interfaces y constantes**  
### 🔴 **Problema:**  
- Las interfaces en C# **deben comenzar con "I"**.  
- Las constantes **deben ir en PascalCase y sin guiones bajos**.  

### 🚫 **Código Incorrecto:**  
```csharp
public interface usuarioRepositorio // ❌ Falta "I" en la interfaz
{
    void Guardar();
}

public const string APP_NAME = "MiApp"; // ❌ Constante con guiones bajos
```

### ✅ **Solución:** Seguir los prefijos estándar  
```csharp
public interface IUsuarioRepositorio // ✅ Interfaces comienzan con "I"
{
    void Guardar();
}

public const string AppName = "MiApp"; // ✅ Constantes en PascalCase
```
✔️ **Reglas básicas:**  
- **Interfaces** → Comienzan con `I` (`IRepositorio`, `IUsuarioService`).  
- **Constantes** → En PascalCase sin `_` (`TiempoMaximo`, `VersionActual`).  

---

## ❌ **Mala Práctica 4: No seguir la convención en nombres de métodos asincrónicos**  
### 🔴 **Problema:**  
- En C#, **los métodos asincrónicos deben terminar en "Async"**.  
- No hacerlo puede **confundir sobre si un método es síncrono o asincrónico**.  

### 🚫 **Código Incorrecto:**  
```csharp
public async Task<List<Usuario>> ObtenerUsuarios()
{
    await Task.Delay(1000);
    return new List<Usuario>();
}
```

### ✅ **Solución:** Agregar "Async" al final del método  
```csharp
public async Task<List<Usuario>> ObtenerUsuariosAsync()
{
    await Task.Delay(1000);
    return new List<Usuario>();
}
```
✔️ **Reglas básicas:**  
- Métodos asincrónicos **siempre terminan en "Async"** (`ObtenerDatosAsync`, `GuardarAsync`).  

---

## ❌ **Mala Práctica 5: Usar prefijos húngaros**  
### 🔴 **Problema:**  
- No es necesario poner `str`, `int`, `bool` en los nombres de variables.  
- C# **ya tiene tipado fuerte**, por lo que es redundante.  

### 🚫 **Código Incorrecto:**  
```csharp
string strNombre = "Juan"; // ❌ "str" es innecesario
int intEdad = 25; // ❌ "int" ya está claro por el tipo de dato
bool blnActivo = true; // ❌ "bln" es redundante
```

### ✅ **Solución:** Usar nombres sin prefijos innecesarios  
```csharp
string nombre = "Juan"; // ✅ Más claro y limpio
int edad = 25; // ✅ Se entiende sin redundancia
bool activo = true; // ✅ No hace falta "bln"
```
✔️ **Reglas básicas:**  
- **Evitar prefijos innecesarios** (`nombre`, no `strNombre`).  

---

## ❌ **Mala Práctica 6: Usar abreviaciones innecesarias**  
### 🔴 **Problema:**  
- Abreviar nombres **demasiado** hace que el código sea difícil de leer.  
- Puede causar confusión si alguien más lo lee.  

### 🚫 **Código Incorrecto:**  
```csharp
var usrLst = ObtenerUsrLst(); // ❌ ¿Qué es "UsrLst"?
var trx = ProcesarTrx(); // ❌ "Trx" es poco claro
```

### ✅ **Solución:** Usar nombres completos  
```csharp
var listaUsuarios = ObtenerListaUsuarios(); // ✅ Más claro
var transaccion = ProcesarTransaccion(); // ✅ Más descriptivo
```
✔️ **Reglas básicas:**  
- **Evitar abreviaciones que no sean estándar** (`usuario`, no `usr`).  
- **Ser explícito en los nombres** (`transaccion`, no `trx`).  

---

## 🎯 **Resumen de Buenas Prácticas en Nomenclatura**  
✅ **PascalCase** para clases, métodos y propiedades (`MiClase`, `ObtenerUsuarios`).  
✅ **camelCase** para variables y parámetros (`nombreUsuario`, `cantidadVentas`).  
✅ **Usar nombres descriptivos**, evitar `x`, `y`, `a1`.  
✅ **Prefijos correctos:** `I` en interfaces, PascalCase en constantes.  
✅ **Métodos asincrónicos terminan en "Async"** (`ObtenerDatosAsync`).  
✅ **No usar prefijos húngaros** (`nombre`, no `strNombre`).  
✅ **Evitar abreviaciones poco claras** (`transaccion`, no `trx`).  

---