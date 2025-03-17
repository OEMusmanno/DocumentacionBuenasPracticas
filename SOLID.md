# 🏗️ **Principios SOLID **  

Los principios SOLID son **cinco reglas fundamentales** que ayudan a escribir código más **mantenible, escalable y flexible** en aplicaciones C#. En un sistema bancario, estos principios aseguran que el código **sea fácil de entender, probar y modificar** sin generar dependencias innecesarias.  

A continuación, te explico **cada principio con ejemplos aplicados al ámbito bancario**.  

---

## 1️⃣ **S – Principio de Responsabilidad Única (SRP – Single Responsibility Principle)**  

### 📌 **Problema**  
Una clase de **CuentaBancaria** maneja tanto **transacciones, impresión de reportes y notificaciones**, lo cual la hace difícil de mantener y probar.  

### 💡 **Solución**  
Cada clase debe **tener una única razón para cambiar**. Debemos separar responsabilidades en diferentes clases.  

### 🔥 **Ejemplo – Mala Práctica**  
```csharp
public class CuentaBancaria
{
    public void Depositar(decimal monto) { /* Lógica de depósito */ }
    public void Retirar(decimal monto) { /* Lógica de retiro */ }
    public void GenerarReporte() { /* Lógica de reportes */ }
    public void EnviarNotificacion() { /* Lógica de notificaciones */ }
}
```
🛑 **Problema**: La clase hace demasiado.  

### ✅ **Solución – Separar Responsabilidades**  
```csharp
public class CuentaBancaria
{
    public void Depositar(decimal monto) { /* Lógica de depósito */ }
    public void Retirar(decimal monto) { /* Lógica de retiro */ }
}

public class ReporteBancario
{
    public void GenerarReporte() { /* Lógica de reportes */ }
}

public class NotificacionServicio
{
    public void EnviarNotificacion() { /* Lógica de notificaciones */ }
}
```
✔ **Beneficio**: Cada clase tiene **una única responsabilidad**, facilitando su mantenimiento.  

---

## 2️⃣ **O – Principio de Abierto/Cerrado (OCP – Open/Closed Principle)**  

### 📌 **Problema**  
Cada vez que agregamos un nuevo tipo de cuenta bancaria, **modificamos** la clase base, aumentando el riesgo de errores.  

### 💡 **Solución**  
Las clases deben estar **abiertas para extensión, pero cerradas para modificación**.  

### 🔥 **Ejemplo – Mala Práctica**  
```csharp
public class CuentaBancaria
{
    public decimal CalcularInteres(string tipoCuenta, decimal saldo)
    {
        if (tipoCuenta == "Ahorro") return saldo * 0.02m;
        if (tipoCuenta == "Corriente") return saldo * 0.01m;
        return 0;
    }
}
```
🛑 **Problema**: Cada nuevo tipo de cuenta **requiere modificar la clase**.  

### ✅ **Solución – Usar Polimorfismo**  
```csharp
public abstract class CuentaBancaria
{
    public abstract decimal CalcularInteres(decimal saldo);
}

public class CuentaAhorro : CuentaBancaria
{
    public override decimal CalcularInteres(decimal saldo) => saldo * 0.02m;
}

public class CuentaCorriente : CuentaBancaria
{
    public override decimal CalcularInteres(decimal saldo) => saldo * 0.01m;
}
```
✔ **Beneficio**: Podemos agregar nuevas cuentas sin modificar la clase original.  

---

## 3️⃣ **L – Principio de Sustitución de Liskov (LSP – Liskov Substitution Principle)**  

### 📌 **Problema**  
Una subclase **no debe alterar el comportamiento** esperado de su clase base.  

### 💡 **Solución**  
Si una clase hija **rompe** el comportamiento de la clase padre, significa que la jerarquía está mal diseñada.  

### 🔥 **Ejemplo – Mala Práctica**  
```csharp
public class CuentaBancaria
{
    public virtual void Retirar(decimal monto) { /* Lógica de retiro */ }
}

public class CuentaPlazoFijo : CuentaBancaria
{
    public override void Retirar(decimal monto)
    {
        throw new InvalidOperationException("No se puede retirar de una cuenta a plazo fijo.");
    }
}
```
🛑 **Problema**: `CuentaPlazoFijo` no debería permitir retiros, pero **hereda** de `CuentaBancaria`, rompiendo el principio LSP.  

### ✅ **Solución – Rediseñar la Jerarquía**  
```csharp
public abstract class CuentaBancaria { }

public class CuentaRetirable : CuentaBancaria
{
    public virtual void Retirar(decimal monto) { /* Lógica de retiro */ }
}

public class CuentaPlazoFijo : CuentaBancaria
{
    // No hereda la funcionalidad de retiro
}
```
✔ **Beneficio**: Se evita la herencia incorrecta.  

---

## 4️⃣ **I – Principio de Segregación de Interfaces (ISP – Interface Segregation Principle)**  

### 📌 **Problema**  
Una interfaz no debe **forzar** a una clase a implementar métodos que no necesita.  

### 💡 **Solución**  
Es mejor **dividir** interfaces grandes en interfaces más pequeñas y especializadas.  

### 🔥 **Ejemplo – Mala Práctica**  
```csharp
public interface ICuenta
{
    void Depositar(decimal monto);
    void Retirar(decimal monto);
    void GenerarReporte();
}
```
🛑 **Problema**: `CuentaPlazoFijo` **no debería implementar** `Retirar()`, pero está obligada.  

### ✅ **Solución – Interfaces más específicas**  
```csharp
public interface ICuenta
{
    void Depositar(decimal monto);
}

public interface ICuentaRetirable
{
    void Retirar(decimal monto);
}
```
✔ **Beneficio**: Cada clase **implementa solo lo que necesita**.  

---

## 5️⃣ **D – Principio de Inversión de Dependencias (DIP – Dependency Inversion Principle)**  

### 📌 **Problema**  
Las clases **dependen directamente de otras clases concretas**, lo que dificulta las pruebas unitarias.  

### 💡 **Solución**  
Las clases deben depender de **abstracciones** (`interfaces`), no de implementaciones concretas.  

### 🔥 **Ejemplo – Mala Práctica**  
```csharp
public class ServicioPago
{
    private NotificacionServicio _notificador = new NotificacionServicio();
    
    public void ProcesarPago()
    {
        // Procesar pago
        _notificador.EnviarNotificacion();
    }
}
```
🛑 **Problema**: `ServicioPago` **depende directamente** de `NotificacionServicio`, lo que dificulta el cambio de sistema de notificación.  

### ✅ **Solución – Usar Inyección de Dependencias**  
```csharp
public class ServicioPago
{
    private readonly INotificacionServicio _notificador;

    public ServicioPago(INotificacionServicio notificador)
    {
        _notificador = notificador;
    }

    public void ProcesarPago()
    {
        // Procesar pago
        _notificador.EnviarNotificacion();
    }
}
```
✔ **Beneficio**: Podemos cambiar la implementación sin modificar `ServicioPago`.  

---

# 🎯 **Conclusión**  
Aplicar **SOLID** en un sistema bancario permite:  

✅ **Código más modular y reutilizable.**  
✅ **Menos dependencias entre clases.**  
✅ **Mayor facilidad de pruebas unitarias.**  
✅ **Facilidad para agregar nuevas funcionalidades sin romper el código existente.**  
