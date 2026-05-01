# Active Directory Lab — Architecture Diagram

Below is a simplified visual diagram of the Active Directory environment deployed in Azure.  
It shows the VM, domain controller, OUs, users, groups, and how everything connects.

                         ┌───────────────────────────────┐
                         │        Microsoft Azure         │
                         │     Resource Group: AD-Lab     │
                         └───────────────────────────────┘
                                      │
                                      │
                         ┌───────────────────────────────┐
                         │  Windows Server 2025 VM        │
                         │  Name: AD-DC01                 │
                         │  Size: Standard_B2s            │
                         │  Role: Domain Controller       │
                         └───────────────────────────────┘
                                      │
                                      │  AD DS + DNS Installed
                                      ▼
                         ┌───────────────────────────────┐
                         │        Active Directory        │
                         │        Domain: lab.local       │
                         └───────────────────────────────┘
                                      │
                                      │
        ┌──────────────────────────────────────────────────────────────────────┐
        │                          Organizational Units                        │
        └──────────────────────────────────────────────────────────────────────┘

                ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
                │     IT       │      │   Finance     │      │      HR       │
                └──────────────┘      └──────────────┘      └──────────────┘
                       │                     │                     │
                       │                     │                     │
               ┌────────────┐       ┌────────────┐       ┌────────────┐
               │ IT_Admins  │       │Finance_Users│       │ HR_Users   │
               └────────────┘       └────────────┘       └────────────┘
                       │                     │                     │
                       │                     │                     │
               ┌────────────┐       ┌────────────┐       ┌────────────┐
               │ alice.chen │       │ bob.patel  │       │carol.jones │
               └────────────┘       └────────────┘       └────────────┘


                ┌──────────────┐
                │    Sales      │
                └──────────────┘
                       │
               ┌────────────┐
               │Sales_Users │
               └────────────┘
                       │
               ┌────────────┐
               │david.smith │
               └────────────┘


                ┌──────────────┐
                │  Computers    │
                └──────────────┘
                       │
                       ▼
               (Workstations join here)



## Summary

This diagram represents:

- Azure-hosted Windows Server VM  
- AD DS + DNS running on the VM  
- Domain: **lab.local**  
- Five Organizational Units  
- Department security groups  
- Four sample users assigned to their groups  
- A Computers OU for domain-joined devices  

This structure mirrors a real enterprise environment and supports identity, security, and automation workflows.
