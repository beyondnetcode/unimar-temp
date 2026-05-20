flowchart LR
    U[Usuarios Web y Movil] --> W[SIS_INTRANET Web\nASP.NET MVC 4 / Web API 4]
    W --> B[Business Layer]
    B --> D[Data Layer]
    D --> DB[SQL Server]
    W --> I[Interfaces]
    I --> S[Servicios WCF / REST]

    style W fill:#f4d35e,stroke:#333,stroke-width:2px
    style DB fill:#9ad1d4,stroke:#333,stroke-width:1px
    style S fill:#ee964b,stroke:#333,stroke-width:1px