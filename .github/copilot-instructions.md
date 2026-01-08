# Copilot instructions para ApiEcommerce

Objetivo: dar al agente la mínima y accionable orientación para ser productivo inmediatamente en este repositorio .NET 8 API.

- **Arquitectura (gran vista):** API Web ASP.NET Core con capas simples: Controllers -> Repository (implementaciones en `Repository/` y contratos en `Repository/IRepository/`) -> EF Core `ApplicationDbContext` (`Data/ApplicationDbContext.cs`). AutoMapper se usa para mapear entre `Models` y `Models/Dtos` (perfiles en `Mapping/`). Las migraciones están en `Migrations/`.

- **Punto de entrada & DI:** revisa `Program.cs` (registra `AddDbContext`, `AddScoped<ICategoryRepository, CategoryRepository>`, `AddAutoMapper`). Ejemplo de patrón: `CreatedAtRoute("GetProduct", new { productId = id }, dto)` en controladores.

- **Patrones de proyecto:**
  - DTOs con sufijos `Create*Dto`, `Update*Dto`, `*Dto` en `Models/Dtos/`.
  - Repositorios devuelven `bool Save()` y usan `Save()` al final de escrituras.
  - Verificaciones explícitas en controladores: uso de `ModelState.AddModelError("CustomError", ...)` y retorno de `BadRequest(ModelState)`.
  - Repositorios usan eager-loading para relaciones: `Include(p => p.Category)`.

- **Workflows de desarrollo descubiertos:**
  - Compilar: `dotnet build` (proyecto raíz `ApiEcommerce.csproj`, target `net8.0`).
  - Ejecutar local: `dotnet run` desde la carpeta del proyecto o usar el perfil en `Properties/launchSettings.json`.
  - Base de datos local con Docker: `docker-compose up -d` (ver `docker-compose.yaml` que levanta SQL Server). Credenciales y puerto expuesto en el compose.
  - Migraciones EF: `dotnet ef migrations add <Name>` y `dotnet ef database update` (proyecto principal es la carpeta raíz). Las migraciones ya están en `Migrations/` con timestamps.

- **Convenciones de código útiles para parches automáticos:**
  - Si añades/renuevas un `Model` debes: 1) añadir/ajustar DTO(s) en `Models/Dtos/`, 2) actualizar/añadir `Mapping/*Profile.cs` con `CreateMap<...>()`, 3) agregar/ajustar métodos en la interfaz `Repository/IRepository/*` y su implementación en `Repository/` y 4) crear una migración EF.
  - Cuando modifiques un método de repositorio que altera datos, respeta el patrón `Save()` y retorna `bool` para indicar éxito.

- **Archivos a revisar primero (ejemplos a copiar):**
  - `Controllers/ProductsController.cs` — ejemplos de validación, status codes y `CreatedAtRoute`.
  - `Repository/ProductRepository.cs` — uso de EF Core, `Include`, `Save()` y búsquedas.
  - `Mapping/ProductProfile.cs` — patrones AutoMapper (ReverseMap, ForMember).
  - `Data/ApplicationDbContext.cs` — DbSets y configuración del contexto.
  - `Program.cs` — registro de servicios DI y Swagger en entorno dev.

- **Integraciones externas y configuración:**
  - Connection string clave: nombre `ConexionSql` (revisar `appsettings.json`/`appsettings.Development.json`).
  - Docker compose ya configura SQL Server; si el agente crea datos de ejemplo, levantar primero el servicio `sql`.
  - No hay provider de autenticación configurado (solo `UseAuthorization()`), por tanto no asumir autenticación aún.

- **Qué evitar / supuestos seguros:**
  - No ejecutar `dotnet ef database update` sin confirmar `ConexionSql` en `appsettings` o sin levantar contenedor SQL (docker-compose).
  - No modificar firmas públicas de controllers sin actualizar rutas nombradas (`Name = "GetProduct"` etc.) usadas por `CreatedAtRoute`.

- **Solicito feedback:** Indica si quieres que incluya snippets concretos (línea/archivo) para reglas de estilo o que auto-implemente plantillas CRUD para un nuevo `Entity` siguiendo las convenciones anteriores.

---
Generado automáticamente — pide ajustes si quieres más detalle en pruebas, CI, o ejemplos de commit/PR.
