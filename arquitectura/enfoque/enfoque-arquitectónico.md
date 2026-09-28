# Enfoque Arquitectónico: Clean Architecture

## 1. Propósito
El enfoque de **Clean Architecture (Arquitectura Limpia)** se aplica para gobernar las dependencias internas del código tanto en el backend como en el frontend Angular. Su objetivo principal es asegurar la **Regla de Dependencia**: el código fuente solo debe apuntar hacia adentro, haciendo que el Dominio sea completamente agnóstico de frameworks, controladores, protocolos de transporte y librerías externas.

## 2. Estructura de Capas y Responsabilidades

| Capa | Responsabilidad | Elementos de Ejemplo |
| :--- | :--- | :--- |
| **Dominio (Entities)** | Modela los conceptos y reglas de negocio esenciales. No tiene dependencias de ningún framework o biblioteca externa. | `Producto`, `Pedido`, `Cliente`, `Value Objects`, `Reglas de Negocio`. |
| **Casos de Uso (Application)** | Orquesta los flujos de la aplicación y define la interacción entre las entidades y los puertos secundarios. | `CrearPedidoUseCase`, `ConfirmarCompraUseCase`, `CancelarPedidoUseCase`. |
| **Adaptadores de Interfaz (Adapters)** | Convierte los datos del formato externo al requerido por los casos de uso y viceversa. | `PedidoController`, `ProductoRepositoryImpl`, `PaymentAdapter`, `DTOs`. |
| **Infraestructura (Frameworks & Drivers)** | Aloja las herramientas técnicas, frameworks web, bases de datos y clientes HTTP. | `Express`, `PostgreSQL`, `Stripe SDK`, `Angular Material`. |

## 3. Diagrama de Enfoque y Dependencias

## Diagrama de arquitectura

```mermaid
flowchart LR

    %% ==========================================
    %% USUARIO
    %% ==========================================

    Usuario["👤 Usuario<br/>(Cliente)"]

    %% ==========================================
    %% MARKETPLACE WEB
    %% ==========================================

    subgraph Marketplace["«aplicación» Marketplace Web<br/>Angular 18 · TypeScript"]

        %% ======================================
        %% PRESENTACIÓN
        %% ======================================

        subgraph Presentacion["PRESENTACIÓN<br/>src/app/presentation/"]

            Catalogo["«componente»<br/><b>CatalogoComponent</b><br/>lista y filtra productos"]

            EstadoCarrito["«servicio de estado»<br/><b>EstadoCarrito</b><br/>signals · sin reglas"]

            Carrito["«componente»<br/><b>CarritoComponent</b><br/>resumen y confirmar compra"]

            App["«componente»<br/><b>AppComponent</b><br/>shell de la aplicación"]

        end

        %% ======================================
        %% APLICACIÓN
        %% ======================================

        subgraph Aplicacion["APLICACIÓN · Casos de uso<br/>src/app/application/"]

            ConsultarCatalogo["«caso de uso»<br/><b>ConsultarCatalogoCasoUso</b><br/>ejecutar()"]

            AgregarCarrito["«caso de uso»<br/><b>AgregarAlCarritoCasoUso</b><br/>ejecutar()"]

            RegistrarCompra["«caso de uso»<br/><b>RegistrarCompraCasoUso</b><br/>ejecutar()"]

        end

        %% ======================================
        %% DOMINIO
        %% ======================================

        subgraph Dominio["DOMINIO · Núcleo<br/>src/app/dominio/"]

            subgraph Modelos["Modelos · Entidades y reglas"]

                Producto["«entidad»<br/><b>Producto</b><br/>stock · categoría · precio"]

                CarritoEntidad["«entidad»<br/><b>Carrito</b><br/>inmutable · subtotal · total"]

                Pedido["«entidad»<br/><b>Pedido</b><br/>estados · total"]

                Precios["«reglas»<br/><b>precios.ts</b><br/>comisión 10% · IGV 18%"]

            end

            subgraph Contratos["Contratos · Puertos"]

                RepositorioProductos["«interfaz»<br/><b>RepositorioProductos</b>"]

                RepositorioPedidos["«interfaz»<br/><b>RepositorioPedidos</b>"]

                ProcesadorPagos["«interfaz»<br/><b>ProcesadorPagos</b>"]

                NotificadorCliente["«interfaz»<br/><b>NotificadorCliente</b>"]

            end

        end

        %% ======================================
        %% INFRAESTRUCTURA
        %% ======================================

        subgraph Infraestructura["INFRAESTRUCTURA<br/>src/app/infraestructura/"]

            ProductosMemoria["«adaptador»<br/><b>RepositorioProductosMemoria</b>"]

            ProductosHttp["«adaptador»<br/><b>RepositorioProductosHttp</b>"]

            PedidosMemoria["«adaptador»<br/><b>RepositorioPedidosMemoria</b>"]

            PagosSimulado["«adaptador»<br/><b>ProcesadorPagosSimulado</b>"]

            NotificadorConsola["«adaptador»<br/><b>NotificadorConsola</b>"]

            NotificadorWhatsApp["«adaptador»<br/><b>NotificadorWhatsApp</b>"]

            Tokens["«Angular DI»<br/><b>tokens.ts</b><br/>InjectionToken por contrato"]

        end

        %% ======================================
        %% RAÍZ DE COMPOSICIÓN
        %% ======================================

        Config["«raíz de composición»<br/><b>app.config.ts</b><br/>useFactory + InjectionToken"]

    end

    %% ==========================================
    %% API REST EXTERNA
    %% ==========================================

    API["«sistema externo»<br/><b>Marketplace API REST</b><br/><br/>
    Backend Node.js · monolito modular<br/><br/>
    /api/productos<br/>
    /api/pedidos<br/>
    /api/autorizacion<br/>
    /api/refreshtokens<br/><br/>
    Integración con WhatsApp"]

    %% ==========================================
    %% NAVEGACIÓN
    %% ==========================================

    Usuario -->|navegador| App

    %% ==========================================
    %% PRESENTACIÓN → APLICACIÓN
    %% ==========================================

    Catalogo -->|invoca| ConsultarCatalogo
    Carrito -->|invoca| AgregarCarrito
    Carrito -->|invoca| RegistrarCompra

    %% ==========================================
    %% APLICACIÓN → DOMINIO
    %% ==========================================

    ConsultarCatalogo -.-> RepositorioProductos
    AgregarCarrito -.-> RepositorioProductos

    RegistrarCompra -.-> RepositorioPedidos
    RegistrarCompra -.-> ProcesadorPagos
    RegistrarCompra -.-> NotificadorCliente

    %% ==========================================
    %% INFRAESTRUCTURA → CONTRATOS
    %% ==========================================

    ProductosMemoria -.->|implementa| RepositorioProductos
    ProductosHttp -.->|implementa| RepositorioProductos

    PedidosMemoria -.->|implementa| RepositorioPedidos

    PagosSimulado -.->|implementa| ProcesadorPagos

    NotificadorConsola -.->|implementa| NotificadorCliente
    NotificadorWhatsApp -.->|implementa| NotificadorCliente

    %% ==========================================
    %% INYECCIÓN DE DEPENDENCIAS
    %% ==========================================

    Config -.->|registra| Tokens

    Tokens -.-> RepositorioProductos
    Tokens -.-> RepositorioPedidos
    Tokens -.-> ProcesadorPagos
    Tokens -.-> NotificadorCliente

    %% ==========================================
    %% COMUNICACIÓN CON API
    %% ==========================================

    ProductosHttp -->|HTTP / JSON| API

    PagosSimulado -->|HTTP / JSON| API

    NotificadorWhatsApp -->|HTTP / JSON| API

    %% ==========================================
    %% ESTILOS
    %% ==========================================

    classDef presentacion fill:#dce9f7,stroke:#5b84b1,color:#111;
    classDef aplicacion fill:#e5f1df,stroke:#78a765,color:#111;
    classDef dominio fill:#fff1c9,stroke:#d6ad42,color:#111;
    classDef infraestructura fill:#eadff2,stroke:#9b76b5,color:#111;
    classDef externo fill:#eeeeee,stroke:#777,color:#111;
    classDef composicion fill:#f5f5f5,stroke:#777,color:#111;

    class Catalogo,EstadoCarrito,Carrito,App presentacion;
    class ConsultarCatalogo,AgregarCarrito,RegistrarCompra aplicacion;
    class Producto,CarritoEntidad,Pedido,Precios,RepositorioProductos,RepositorioPedidos,ProcesadorPagos,NotificadorCliente dominio;
    class ProductosMemoria,ProductosHttp,PedidosMemoria,PagosSimulado,NotificadorConsola,NotificadorWhatsApp,Tokens infraestructura;
    class Usuario,API externo;
    class Config composicion;
```

## 4. Reglas de Dependencia del Código
* **Aislamiento del Dominio:** Las entidades no deben contener anotaciones, decoradores de ORM ni tipos de datos propios de Angular o Express.
* **Inversión de Control (DIP):** Los Casos de Uso definen interfaces (puertos) para la persistencia o los servicios externos; las capas externas se encargan de implementar dichas interfaces (adaptadores).
* **Testabilidad:** Los Casos de Uso y las Entidades se prueban de manera unitaria y rápida sin necesidad de levantar bases de datos ni servidores web.