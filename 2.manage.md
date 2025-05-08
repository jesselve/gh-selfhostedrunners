# Manage self-hosted runners using groups

> To control access to runners at the organization level, organizations using the GitHub Team plan can use runner groups. Runner groups are used to collect sets of runners and create a security boundary around them.

See: [Managing access to self-hosted runners using groups](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/managing-access-to-self-hosted-runners-using-groups)

```mermaid
flowchart LR
    %% Organization Level
    subgraph Org["Organization"]
        OrgAdmin["Ledcor GitHub Org Admin
        fa:fa-user-tie"]
        
        OrgManager["Manager with
        'Manage org runners & groups' Permission
        fa:fa-user-cog"]
        
        subgraph OrgRunnerGroups["Organization Runner Groups"]
            OrgDefaultGroup["Default Runner Group
            (All Private Repos)"]
            DataAnalyticsGroup["Data and Analytics Runner Group
            (Selected Repos)"]
        end
        
        subgraph OrgRunners["Organization Runners"]
            DefaultLinuxRunner["Default Runner 1: Linux VM
            Labels: self-hosted, linux"]
            DefaultWinRunner["Default Runner 2: Windows VM
            Labels: self-hosted, windows"]
            DataRunner["Data Runner: Linux VM
            Labels: self-hosted, linux, data-analytics"]
        end
        
        %% Repositories
        subgraph Repos["Private Ledcor Repositories"]
            subgraph RepoA["CS Repo"]
                RepoAWorkflow["GitHub Action Workflow
                runs-on: self-hosted, linux"]
            end
            
            subgraph RepoB["Ops Repo"]
                RepoBWorkflow["GitHub Action Workflow
                runs-on: self-hosted, windows"]
            end
            
            subgraph RepoC["D & A Repo"]
                RepoCWorkflow["GitHub Action Workflow
                runs-on: self-hosted, linux, data-analytics"]
            end
        end
        
        %% Admin connections
        OrgAdmin -->|"Full control over"| OrgRunnerGroups
        OrgAdmin -->|"Delegates permission to"| OrgManager
        
        %% Manager connections
        OrgManager -->|"Can manage runners & groups"| OrgRunnerGroups
        OrgManager -->|"Can register/remove"| OrgRunners
        OrgManager -->|"Can set access for"| Repos
        
        %% Runner Group assignment
        DefaultLinuxRunner -->|"Member of"| OrgDefaultGroup
        DefaultWinRunner -->|"Member of"| OrgDefaultGroup
        DataRunner -->|"Member of"| DataAnalyticsGroup
        
        %% Runner Group access to repositories
        DataAnalyticsGroup -->|"Access granted to"| RepoC
        OrgDefaultGroup -->|"Access granted to all private repos"| RepoA
        OrgDefaultGroup -->|"Access granted to all private repos"| RepoB
        OrgDefaultGroup -->|"Access granted to all private repos"| RepoC
    end

    %% Class Definitions
    classDef org fill:#4B77BE,color:white,stroke:#3A67AE,stroke-width:2px
    classDef orgAdmin fill:#7D97AD,color:white,stroke:#5C768C,stroke-width:2px
    classDef orgManager fill:#5D8AC3,color:white,stroke:#476C9C,stroke-width:2px
    classDef orgGroups fill:#9DB2C5,color:black,stroke:#7C91A4,stroke-width:2px
    classDef orgRunners fill:#ADBFCE,color:black,stroke:#8C9EAD,stroke-width:2px
    classDef repos fill:#2ECC71,color:white,stroke:#27AE60,stroke-width:2px
    classDef repo fill:#F39C12,color:white,stroke:#D35400,stroke-width:2px
    classDef workflow fill:#1ABC9C,color:white,stroke:#16A085,stroke-width:2px
    
    class Org org
    class OrgAdmin orgAdmin
    class OrgManager orgManager
    class OrgRunnerGroups,OrgDefaultGroup,DataAnalyticsGroup orgGroups
    class OrgRunners,DefaultLinuxRunner,DefaultWinRunner,DataRunner orgRunners
    class Repos repos
    class RepoA,RepoB,RepoC repo
    class RepoAWorkflow,RepoBWorkflow,RepoCWorkflow workflow
```
