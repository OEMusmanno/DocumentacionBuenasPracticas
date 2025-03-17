## 🏦 Patrones de Diseño en un Banco  

Los patrones de diseño ayudan a mejorar la arquitectura de un sistema bancario, haciendo que el código sea más **modular, escalable y mantenible**.  

### 1️⃣ **Singleton – Servicio de Autenticación**  
**Problema:** Solo debe existir una instancia del servicio de autenticación en toda la aplicación.  
**Solución:** Usar un Singleton para garantizar una única instancia.  

```csharp
public sealed class ServicioAutenticacion
{
    private static readonly ServicioAutenticacion _instancia = new ServicioAutenticacion();
    public static ServicioAutenticacion Instancia => _instancia;

    private ServicioAutenticacion() { }

    public bool Autenticar(string usuario, string contraseña) => usuario == "admin" && contraseña == "1234";
}
```

🔹 **Mal uso:** Puede dificultar pruebas unitarias por el acoplamiento global.  
🔹 **Mejor uso:** Inyectarlo como dependencia en lugar de llamarlo directamente.  

---

### 2️⃣ **Factory Method – Creación de Cuentas Bancarias**  
**Problema:** Se deben crear distintos tipos de cuentas (corriente, ahorro) sin acoplar el código a clases específicas.  
**Solución:** Usar un método fábrica.  

```csharp
public abstract class CuentaBancariaFactory
{
    public abstract ICuentaBancaria CrearCuenta();
}

public class CuentaAhorroFactory : CuentaBancariaFactory
{
    public override ICuentaBancaria CrearCuenta() => new CuentaAhorro();
}
```

🔹 **Beneficio:** Permite agregar nuevos tipos de cuentas sin modificar el código existente.  

---

### 3️⃣ **Abstract Factory – Diferentes Tipos de Tarjetas**  
**Problema:** Queremos crear familias de productos relacionados, como tarjetas de débito y crédito.  
**Solución:** Usar una **Abstract Factory**.  

```csharp
public interface IBancoFactory
{
    ITarjetaCredito CrearTarjetaCredito();
    ITarjetaDebito CrearTarjetaDebito();
}

public class BancoXFactory : IBancoFactory
{
    public ITarjetaCredito CrearTarjetaCredito() => new TarjetaCreditoVisa();
    public ITarjetaDebito CrearTarjetaDebito() => new TarjetaDebitoVisa();
}
```

🔹 **Beneficio:** Se pueden cambiar familias de productos sin modificar el código cliente.  

---

### 4️⃣ **Builder – Generación de Reportes Bancarios**  
**Problema:** Un reporte bancario tiene muchos parámetros opcionales.  
**Solución:** Usar un Builder para construir el reporte paso a paso.  

```csharp
public class ReporteBancarioBuilder
{
    private ReporteBancario _reporte = new ReporteBancario();

    public ReporteBancarioBuilder ConSaldo(decimal saldo)
    {
        _reporte.Saldo = saldo;
        return this;
    }

    public ReporteBancario Build() => _reporte;
}
```

🔹 **Ventaja:** Evita múltiples constructores con demasiados parámetros.  

---

### 5️⃣ **Prototype – Clonación de Clientes**  
**Problema:** Queremos copiar datos de un cliente sin modificar el original.  
**Solución:** Implementar `ICloneable`.  

```csharp
public class Cliente : ICloneable
{
    public string Nombre { get; set; }
    public object Clone() => new Cliente { Nombre = this.Nombre };
}
```

🔹 **Cuándo usarlo:** Cuando copiar un objeto es más eficiente que crearlo desde cero.  

---

### 6️⃣ **Adapter – Integración con APIs de Otros Bancos**  
**Problema:** Nuestro sistema usa una API bancaria diferente a la que necesitamos integrar.  
**Solución:** Usar un **adaptador**.  

```csharp
public class AdaptadorBanco : INuevaAPI
{
    private APIBancoViejo _bancoViejo = new APIBancoViejo();
    public void ProcesarPago() => _bancoViejo.RealizarTransaccion();
}
```

🔹 **Ejemplo real:** Adaptar una API de **Visa** para que funcione con nuestro sistema de pagos.  

---

### 7️⃣ **Decorator – Seguridad en Transacciones**  
**Problema:** Queremos agregar validaciones de seguridad sin modificar el código base.  
**Solución:** Usar un **decorador**.  

```csharp
public class TransaccionSegura : ITransaccion
{
    private ITransaccion _transaccion;

    public TransaccionSegura(ITransaccion transaccion)
    {
        _transaccion = transaccion;
    }

    public void Ejecutar()
    {
        VerificarAutenticacion();
        _transaccion.Ejecutar();
    }

    private void VerificarAutenticacion() => Console.WriteLine("Autenticación válida.");
}
```

🔹 **Cuándo usarlo:** Para agregar seguridad sin modificar clases existentes.  

---

### 8️⃣ **Observer – Notificaciones de Movimientos Bancarios**  
**Problema:** Cuando ocurre un depósito o retiro, queremos notificar a los clientes.  
**Solución:** Implementar un patrón **Observer**.  

```csharp
public class CuentaBancaria
{
    private List<ICliente> _clientes = new();

    public void AgregarCliente(ICliente cliente) => _clientes.Add(cliente);

    public void Depositar(decimal monto)
    {
        Console.WriteLine($"Depósito de {monto}");
        Notificar();
    }

    private void Notificar() => _clientes.ForEach(c => c.RecibirNotificacion());
}
```

🔹 **Ejemplo real:** Notificaciones por SMS cuando se hace una compra con tarjeta.  

---

### 9️⃣ **Mediator – Comunicación entre Servicios Bancarios**  
**Problema:** Un servicio de pagos y uno de validación deben comunicarse sin estar acoplados.  
**Solución:** Usar un **mediador**.  

```csharp
public class MediadorBanco : IMediador
{
    private List<IServicioBanco> _servicios = new();
    
    public void Registrar(IServicioBanco servicio) => _servicios.Add(servicio);

    public void EnviarMensaje(string mensaje) => _servicios.ForEach(s => s.RecibirMensaje(mensaje));
}
```

🔹 **Ejemplo real:** **MediatR** en CQRS para separar lógica de negocio y capa de datos.  

---

### 🔟 **Strategy – Cálculo de Intereses según Tipo de Cuenta**  
**Problema:** Cada cuenta tiene una fórmula diferente para calcular intereses.  
**Solución:** Usar el **patrón Strategy**.  

```csharp
public interface ICalculoInteres
{
    decimal Calcular(decimal saldo);
}

public class InteresCuentaAhorro : ICalculoInteres
{
    public decimal Calcular(decimal saldo) => saldo * 0.02m;
}
```

🔹 **Cuándo usarlo:** Cuando hay múltiples estrategias para un mismo proceso (como tasas de interés variables).  

---

## 📌 Conclusión  

Aplicar patrones de diseño en un **sistema bancario** mejora la organización del código, permitiendo que sea más flexible y fácil de mantener. 🚀  