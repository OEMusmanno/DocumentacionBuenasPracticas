# 📌 Buenas Prácticas en C#

Esta documentación reúne las mejores prácticas para desarrollar en C#, enfocándose en código limpio, eficiente y mantenible. Se abordan aspectos clave como **asincronismo**, **conexión a bases de datos**, **patrones de diseño** y **nomenclatura** y **mucho mas!**.

---

## 📖 Contenido  

🔹 **[Principios SOLID](solid.md)**  
Los principios SOLID son **buenas prácticas de diseño orientado a objetos** que mejoran la **escalabilidad, mantenibilidad y flexibilidad del código**.  

**Temas que se tocan**  

✅ **S - Single Responsibility Principle (SRP)**: Cada clase debe tener una única responsabilidad.  
✅ **O - Open/Closed Principle (OCP)**: El código debe estar abierto a extensiones, pero cerrado a modificaciones.  
✅ **L - Liskov Substitution Principle (LSP)**: Las clases derivadas deben poder sustituir a sus clases base sin romper la funcionalidad.  
✅ **I - Interface Segregation Principle (ISP)**: No forzar a los clientes a depender de interfaces que no usan.  
✅ **D - Dependency Inversion Principle (DIP)**: Depender de abstracciones y no de implementaciones concretas.  

---

🔹 **[Asincronismo](asincronismo.md)**  
El asincronismo en C# es clave para mejorar la **eficiencia y rendimiento** de las aplicaciones. Sin embargo, un mal uso puede generar **bloqueos, deadlocks y problemas de escalabilidad**.  

**Temas que se tocan:**  
✅ Usar `await` en toda la cadena de llamadas para evitar bloqueos.  
✅ Evitar `.Result`, `.Wait()` y `Task.Run()` innecesarios.  
✅ Ejecutar tareas en paralelo con `Task.WhenAll`.  
✅ Manejar excepciones con `try-catch`.  
✅ Evitar `async void`, usar siempre `async Task`.  

---  
🔹 **[Uso Correcto de HttpClient](httpclient.md)**  
El uso incorrecto de `HttpClient` puede generar **fugas de memoria, bloqueos y saturación del servidor**.  

**Temas que se tocan**  

✅ Diferencias entre `HttpClient` y `HttpClientFactory`.  
✅ Configuración de `HttpClientFactory` en .NET.  
✅ Evitar la creación innecesaria de instancias de `HttpClient`.  
✅ Uso de `IHttpClientFactory` para gestionar conexiones.  
✅ Configuración de `Timeout`, `MaxConnectionsPerServer` y `HandlerLifetime`.   

---

🔹 **[Conexión a Base de Datos](conexion_bd.md)**  
Una correcta gestión de la conexión a base de datos **optimiza el rendimiento y evita fugas de recursos**.  

**Temas que se tocan:**  
✅ Usar `using` para cerrar conexiones correctamente.  
✅ Configurar correctamente `Max Pool Size` y `Connection Lifetime`.  
✅ Usar `DbContext` correctamente en Entity Framework Core.  
✅ Evitar mantener conexiones abiertas innecesariamente.  
✅ Implementar el patrón **Repositorio y Unidad de Trabajo**.  

---  

🔹 **[Patrones de Diseño](patrones_diseno.md)**  
Los patrones de diseño ayudan a estructurar el código para que sea **modular, reutilizable y fácil de mantener**.  

**Temas que se tocan:**  
✅ Aplicar el patrón **Repository** para acceder a datos.  
✅ Usar **Factory Pattern** para evitar instancias innecesarias.  
✅ Aplicar **Singleton** con precaución para evitar estados globales.  
✅ Implementar **Mediator** para reducir el acoplamiento entre servicios.  
✅ Utilizar **Decorator** para extender funcionalidades sin modificar código existente.  

---  

🔹 **[Nomenclatura](nomenclatura.md)**  
Seguir una nomenclatura clara y coherente mejora la **legibilidad y mantenibilidad** del código.  

**Temas que se tocan:**  
✅ Usar `PascalCase` para nombres de clases y métodos.  
✅ Utilizar `camelCase` para variables y parámetros.  
✅ Evitar nombres genéricos como `temp`, `data`, `obj`.  
✅ Prefijar interfaces con `I`, ejemplo: `IUsuarioService`.  
✅ Seguir convenciones de Microsoft en nombres de propiedades y métodos.  

---  

🔹 **[Inyección de Dependencias](inyeccion_dependencias.md)**  
Una correcta inyección de dependencias facilita la **modularidad, testabilidad y escalabilidad** del código.  

**Temas que se tocan:**  
✅ Evitar `new` dentro de los controladores, usar inyección de constructor.  
✅ No usar clases estáticas para servicios inyectables.  
✅ Registrar correctamente dependencias en `Program.cs`.  
✅ No usar `Singleton` para servicios con estado.  
✅ Aplicar `Facade` o `Mediator` si hay muchas dependencias en un constructor.  

---

Cada sección contiene ejemplos de **buenas y malas prácticas**, explicaciones detalladas y código optimizado. 
