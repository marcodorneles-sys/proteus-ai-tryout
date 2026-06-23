---
status: proposed
---

# Core Technology & Runtime Stack

To replace the legacy SAP MII GSCA Machinery system (losing support in 2027) and eliminate untestable monolithic stored procedures, we are adopting a decoupled architecture to accelerate developer velocity. We will use React for the frontend, .NET/ASP.NET Core (exact LTS version pending JTI engineering standards confirmation) for the backend, Azure SQL Database for the data layer, and Azure Container Apps as our primary runtime.

## Considered Options
* **Blazor (Frontend):** Rejected. While Blazor is a natural fit for a .NET backend, AI coding agents like GitHub Copilot are significantly more proficient with React. React was chosen explicitly to support the project's "AI-First" delivery goals.

## Module Reference
This adr refgrences **Proteus Architecture Document:**
# path: _docs\architecture\proteus_architecture_document.md