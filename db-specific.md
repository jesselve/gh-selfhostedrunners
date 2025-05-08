# Databricks-specific GitHub self-hosted runner recommendation

## Notes

Given the volume and complexity of the network requirements for Databricks in Azure, it is recommended to co-locate a dedicated GitHub self-hosted runner ***within*** the Databrick VNet for this use case.

```mermaid
flowchart TB
    %% Main VNet Components
    subgraph DatabricksVNet["Azure VNet for Databricks"]
        RunnerNSG["Runner Network Security Group (NSG)"]
        DatabricksNSG["Databricks Network Security Group (NSG)"]
        
        subgraph RunnerSubnet["GitHub Runner Subnet"]
            GitHubRunner["Self-hosted Runner VM
            fa:fa-cogs"]
            
            ManagedIdentity["Azure Managed Identity
            fa:fa-id-badge"]
            
            GitHubRunner -->|"Uses"| ManagedIdentity
        end
        
        subgraph DatabricksSubnets["Databricks Subnets"]
            subgraph PublicSubnet["Public/Host Subnet"]
                DBHost["Databricks Host VM
                fa:fa-server"]
            end
            
            subgraph PrivateSubnet["Private/Container Subnet"]
                DBContainer["Databricks Container
                fa:fa-box"]
            end
        end
        
        %% Direct Communication within VNet
        GitHubRunner -->|"Direct deployment via Databricks API
        (Internal VNet traffic)"| DatabricksSubnets
    end
    
    %% External Services
    subgraph ExternalServices["External Services"]
        GHServices["GitHub Services
        fa:fa-github"]
        
        subgraph AzureDatabricksControlPlane["Azure Databricks Control Plane (Microsoft Managed)"]
            APIService["REST API Service
            fa:fa-cogs"]
            RelayService["Secure Cluster Connectivity Relay
            fa:fa-exchange-alt"]
        end
    end
    
    %% Storage and Resources
    subgraph AzureStorage["Azure Storage & Resources"]
        ADLS["Azure Data Lake Storage
        fa:fa-database"]
        KeyVault["Azure Key Vault
        fa:fa-key"]
    end
    
    %% Private Endpoints for Storage
    PrivateEndpoint1["Private Endpoint for ADLS"] --> ADLS
    PrivateEndpoint2["Private Endpoint for KeyVault"] --> KeyVault
    
    DatabricksVNet --> PrivateEndpoint1
    DatabricksVNet --> PrivateEndpoint2
    
    %% Required Outbound Access
    GitHubRunner -->|"HTTPS (TCP Port 443)
    To GitHub Services"| GHServices
    
    GitHubRunner -->|"HTTPS (TCP Port 443)
    For API Access"| APIService
    
    %% Databricks communication with Control Plane
    DBContainer -->|"HTTPS (Port 443)
    Long Poll Connection"| RelayService
    DBHost -->|"HTTPS (Port 443)"| APIService
    
    %% NSG Rules
    RunnerNSG -->|"Outbound Rules:
    - Allow to GitHub Services (see docs for full list)
    - Allow to Azure Services (e.g., *.blob.core.windows.net, Key Vault, ADLS)
    - Deny All Other Traffic"| GitHubRunner
    
    DatabricksNSG -->|"Inbound/Outbound Rules for Databricks:
    - Standard Databricks NSG Requirements"| DatabricksSubnets
    
    %% Class Definitions
    classDef vnet fill:#0078D4,color:white,stroke:#005BA1,stroke-width:2px
    classDef runner fill:#E95420,color:white,stroke:#C33C12,stroke-width:2px
    classDef databricksSub fill:#50A7F0,color:black,stroke:#3991DA,stroke-width:2px
    classDef databricksRes fill:#83C3F7,color:black,stroke:#5DA7E8,stroke-width:2px
    classDef external fill:#24292E,color:white,stroke:#1B1F23,stroke-width:2px
    classDef controlPlane fill:#3C873A,color:white,stroke:#2D682B,stroke-width:2px
    classDef storage fill:#FFD700,color:black,stroke:#DAA520,stroke-width:2px
    classDef nsg fill:#E74C3C,color:white,stroke:#C0392B,stroke-width:2px
    classDef endpoint fill:#9B59B6,color:white,stroke:#8E44AD,stroke-width:2px
    classDef identity fill:#1ABC9C,color:white,stroke:#16A085,stroke-width:2px
    
    class DatabricksVNet vnet
    class RunnerSubnet runner
    class GitHubRunner runner
    class ManagedIdentity identity
    class DatabricksSubnets,PublicSubnet,PrivateSubnet databricksSub
    class DBHost,DBContainer databricksRes
    class GHServices external
    class AzureDatabricksControlPlane,APIService,RelayService controlPlane
    class AzureStorage,ADLS,KeyVault storage
    class RunnerNSG,DatabricksNSG nsg
    class PrivateEndpoint1,PrivateEndpoint2 endpoint
```
