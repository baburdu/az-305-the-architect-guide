# **AZ-305: The Architect's Definitive Guide**

##

## **🏛️ Volume I: Identity, Authentication, and Governance**

Identity is the primary security perimeter of the modern cloud. In the AZ-305 curriculum, this domain accounts for approximately 30% of the exam questions.

### **1.1 The Azure Organizational Hierarchy and Scope**

Designing for an enterprise requires a rigid understanding of how management layers propagate policies and permissions. The **Scope** defines the level at which an assignment, policy, or lock is applied.

* **(L1) Tenants (Microsoft Entra ID)**: The top-level logical boundary that represents your organization and houses all identities. This is the ultimate "Root" for authentication.

* **(L2) Management Groups (MGs)**: These are containers designed to govern multiple subscriptions. Any policy applied here is inherited by all subscriptions within the group, making it the ideal spot for enterprise-wide compliance.

* **(L3) Subscriptions**: The primary boundary for billing and resource quotas. It acts as a logical partition for resources.

* **(L4) Resource Groups (RGs)**: Logical containers for grouping resources that share a common lifecycle.

* **(L5) Resources**: The granular level for individual service instances, such as VMs, Storage Accounts, or Databases.

#### **🔬 Architectural Inheritance & Governance Logic**

* **The Inheritance Rule**: Due to the hierarchy, do know what can be inherited and what will not. For example, an Azure Policy applied at the Tenant level will impact every lower level in the hierarchy.

* **Resource Locks**: A "CanNotDelete" or "ReadOnly" lock applied at the Resource Group level will impact every single child Resource within it.

* **Policy vs. RBAC**: Policies focus on resource properties (e.g., "Must be in East US"), while RBAC (Role-Based Access Control) focuses on user actions (e.g., "Can create a VM").

### 1.2 Microsoft Entra ID Governance

Expect at least 3-5 questions focused specifically on Entra ID Governance. This section is about the automation of access and the security of "Identity Lifecycles."

#### **🛡️ Privileged Identity Management (PIM)**

PIM is the "Hero" service for reducing the attack surface. It provides **Just-in-Time (JIT)** privileged access.

* **Activation Requirement**: Users do not have permanent admin rights. They must "Activate" their role, providing a justification and potentially undergoing MFA.

* **Approval Workflows**: You can configure roles that require an designated approver to authorize the request before access is granted.  
* **History and Auditing**: Every activation is logged, ensuring that you can audit who had "Owner" or "Global Admin" rights at any specific time.

#### **🔄 Access Reviews**

This is an automated process to certify that users still require their current permissions.

* **The Problem**: "Permission Creep"—where users accumulate access as they move through different roles in a company.  
* **The Solution**: Periodic reviews where a manager or the user themselves must confirm they still need access. If they fail to respond, access can be automatically revoked based on policy.

#### **🤖 Lifecycle Workflows**

These automate the "Joiner, Mover, Leaver" process.

* **Workforce Management**: You can integrate with HR systems (like Workday or SAP SuccessFactors) to automate the creation and removal of users.

* **Automation**: When an HR system marks an employee as "terminated," the Lifecycle Workflow can automatically disable the Entra ID account, revoke their sessions, and remove them from all groups.

### 1.3 Hybrid Identity Architectures

Connecting on-premises directories to the cloud is a "HUGE" topic for the exam.

#### **🔄 Entra Connect Sync vs. Cloud Sync**

* **Entra Connect Sync**: The robust, traditional tool installed on an on-premises server. It is preferred for complex environments requiring attribute filtering or password writeback for legacy setups.

* **Entra Cloud Sync**: A lightweight, agent-based alternative designed for multi-forest or disconnected environments. It is managed primarily from the cloud, reducing on-prem footprint.

#### **🔑 Authentication Flow Comparison**

* **Pass-through Authentication (PTA)**: Password validation occurs strictly on-premises. This is used by organizations that cannot store even a hash of a password in the cloud for compliance reasons.

* **Password Hash Sync (PHS)**: A hash of the password hash is synchronized to Entra ID. This is the simplest to manage and enables "Leaked Credential Detection."

* **Federation (AD FS)**: Relies on an external identity provider (like on-prem AD FS) for authentication.

* **Entra Domain Services (AD DS)**: Provides managed domain services (Domain Join, Group Policy, LDAP, Kerberos) in the cloud for legacy applications that cannot use modern auth.

### 1.4 Licensing, Risk, and Security

Choosing the right license is a major part of the AZ-305 architect's job.

📊 The License Matrix

| Feature | Entra ID Free | P1 (E3) | P2 / Governance (E5) |
| :---- | :---- | :---- | :---- |
| **SSO & MFA** | Basic | Included | Included |
| **Conditional Access** | No | Yes | Yes |
| **PIM & Access Reviews** | No | No | Yes |
| **Identity Protection** | No | No | Yes |

🚨 Identity Protection (1-3 Questions)

* **User Risk**: The probability that a user's identity has been compromised (e.g., credentials found on the dark web).

* **Sign-in Risk**: The probability that a specific authentication request is suspicious (e.g., an "Impossible Travel" sign-in from two different countries).

* **Conditional Access**: This is the "If-Then" engine of Entra ID. **Example**: *If* a user is in the Finance group and the Sign-in Risk is *Medium*, *Then* require MFA and a Managed Device.

## **🛠️ Volume II: Data Engineering and Modern Integration**

*Architecting how data is orchestrated, transformed, and communicated is a core pillar of the AZ-305 exam. This section focuses on the distinction between purely analytical services and orchestration engines.*

### **2.1 The Data Orchestration and Analytics Layer**

When designing a data solution, the choice between Azure Synapse and Azure Data Factory (ADF) is critical. In your architectural designs, consider whether you need a specialized tool for movement or an integrated environment for the entire lifecycle.

#### **🏛️ Azure Synapse Analytics**

Azure Synapse is described as an "all-in-one shop" service, specifically designed for large-scale data warehousing and Big Data processing.

* **Unified Experience**: It brings together SQL technologies used in enterprise data warehousing, Spark technologies used for big data, and Data Explorer for log and time-series analytics.  
* **Serverless and Dedicated Tiers**: You can choose between serverless SQL pools for unplanned or ad-hoc workloads and dedicated SQL pools for high-performance, predictable data warehousing.  
* **Synapse Pipelines**: While Synapse has built-in orchestration, it is essentially the same engine found in Azure Data Factory, integrated into a single workspace.

#### **🔄 Azure Data Factory (ADF)**

Azure Data Factory is a "pure orchestrator" for data engineering pipelines. It is used to create, schedule, and orchestrate your ETL (Extract, Transform, Load) and ELT (Extract, Load, Transform) workflows.

* **No-Code Transformation**: Use Mapping Data Flows to transform data at scale without writing code.  
* **Connectivity**: It features over 90+ built-in connectors to pull data from diverse sources, including SaaS apps, on-premises databases, and other clouds.  
* **The Integration Runtime (IR)**: This is the "compute engine" used by ADF to perform data movement and transformation activities.  
  * **Self-hosted IR**: This is mandatory when your pipeline needs to reach data residing in an on-premises network or a virtual network with private access.

---

### **2.2 Data Migration and Offline Mechanisms**

Architects must decide how to physically move data into Azure, especially when bandwidth is a constraint or when dealing with legacy file systems.

#### **📦 The Data Box Family**

For large-scale, offline migrations where the network is not a viable option, use the **Azure Data Box Family** (Microsoft-provided hardware).

* **Data Box Disk**: Small, portable SSDs for up to 40 TB.  
* **Data Box**: A ruggedized device for up to 100 TB.  
* **Data Box Heavy**: A massive, luggable device for up to 1 PB of data.

#### **🚢 Azure Import/Export**

Unlike the Data Box family, the **Azure Import/Export** service requires you to use your own physical disks to ship data to or from an Azure datacenter. This is often used for smaller datasets or when specific hardware security requirements must be met.

#### **🛠️ AzCopy and CLI Tools**

* **AzCopy**: A high-performance command-line utility used specifically for copying or moving Blobs and files to and from storage accounts.  
* **Versatility**: While you don't need to memorize exact command syntax for the exam, you must know that it can be leveraged via API, CLI, and az commands to automate data movement.

#### **📂 Azure File Sync**

This service allows you to centralize your organization's file shares in Azure Files while maintaining the local performance of an on-premises file server.

* **Tiering**: It "tiers" infrequently accessed files to the cloud while keeping the most frequently used data on the local server.  
* **Disaster Recovery**: If your local server fails, you can quickly set up a new one and point it to the Azure File share, restoring access through the local cache.

---

### **2.3 Messaging and Event-Driven Architecture**

Knowing when to use which messaging service is a mandatory skill for the AZ-305 exam. You must distinguish between "intent-based" messaging and "event-based" routing.

#### **📧 Azure Service Bus**

A high-reliability enterprise messaging service used for complex communication patterns.

* **Queues**: Used for 1:1 "point-to-point" communication where a message is sent to a single receiver and processed once.  
* **Topics and Subscriptions**: Used for 1:Many "publish/subscribe" models. One message is sent to a topic and can be received by multiple subscribers based on filter rules.  
* **Best For**: Financial transactions, order processing, and scenarios requiring high reliability and stateful messaging.

#### **⚡ Azure Event Grid**

A reactive, event-driven routing service that enables you to build applications with "serverless" logic.

* **Logic**: "Something happened (Event) \-\> Tell someone about it (Handler)".  
* **Example**: A file is uploaded to a Storage Account (Event), which triggers an Azure Function to process that file (Handler).

#### **📊 Azure Event Hubs**

A big data streaming platform and event ingestion service.

* **Scale**: It can receive and process millions of events per second.  
* **Best For**: Telemetry, application logging, and real-time analytics pipelines.

#### **⚙️ Logic Apps vs. Azure Functions**

* **Logic Apps**: A low-code/no-code service for "Workflow & Orchestration". Use this when you need to connect hundreds of SaaS services together via a visual designer.  
* **Azure Functions**: A "Serverless Compute" service for running small pieces of code (functions) in the cloud. Use this for complex logic, data processing, or when you need the flexibility of a programming language.

---

### **2.4 Security and Visualization**

* **Azure Key Vault**: As an architect, you must know that the mandatory role of Key Vault is to provide a secure, centralized location for secrets, keys, and certificates. You are not required to know *how* it works internally for the exam, but rather *what* it is for.  
* **Power BI**: Serves as the primary viewer and dashboard tool for data visualization, turning your Synapse or SQL data into actionable insights.

## **💾 Volume III: Data Storage and SQL Architectures**

The heart of any Azure solution lies in its data persistence layer. For the AZ-305 exam, storage and SQL design account for a massive 20-25% of the total content. Architects must be able to choose the correct storage account, replication strategy, and SQL tier to meet specific performance and cost requirements.

---

### **3.1 Storage Account Fundamentals**

Selecting the correct account type is the foundation of your storage architecture. Your design decisions here directly impact which features (like NFS support or Blob versioning) are available.

#### **📂 Account Type Deep-Dive**

* **General Purpose v2 (GPv2)**: The modern standard and "heart" of Azure Storage. It supports Blobs, Files, Queues, and Tables. Use this for almost all standard workloads unless specific Premium performance is required.  
* **Premium Block Blob Storage**: Optimized for high transaction rates or scenarios requiring very low, consistent storage latency.  
* **Premium File Storage**: Used specifically for high-performance enterprise file shares, supporting both SMB and NFS protocols.  
* **Legacy Types**: While GPv1 and BlobStorage accounts still exist, they are rarely the correct architectural answer for new deployments.

#### **❄️ Blob Access Tiers**

Architects must optimize for cost by matching data access patterns to the correct tier.

* **Hot Tier**: Optimized for data that is accessed frequently.  
* **Cool Tier**: For data accessed infrequently and stored for at least 30 days.  
* **Cold Tier**: For data accessed rarely and stored for at least 90 days.  
* **Archive Tier**: The lowest cost for long-term storage (180+ days). Data is "offline" and requires a rehydration process that can take hours.

---

### 3.2 Replication and Resiliency

It is **IMPERATIVE** to know the 6 replication types and exactly how many copies of data each creates to ensure business continuity.

| Replication Type | Copies | Strategy | Resiliency Level |
| :---- | :---- | :---- | :---- |
| **LRS** (Locally Redundant) | 3 | Single Datacenter | Protects against disk/rack failure. |
| **ZRS** (Zone Redundant) | 3 | 3 Availability Zones | Protects against a full Datacenter outage. |
| **GRS** (Geo-Redundant) | 6 | 3 Local \+ 3 in Paired Region | Protects against a Regional disaster. |
| **RA-GRS** | 6 | Same as GRS | Adds Read-only access to the secondary region. |
| **GZRS** | 6 | 3 Zonal \+ 3 in Paired Region | Highest availability; protects against Zone \+ Region failure. |
| **RA-GZRS** | 6 | Same as GZRS | Adds Read-only access to the geo-secondary region. |

---

### 3.3 SQL Solutions and Migration Strategy

Azure SQL architecture is as heavy as Identity in the exam. You must be able to distinguish between the three primary hosting models based on the level of administrative control and compatibility required.

#### **🏗️ The SQL Hosting Matrix**

* **Azure SQL Database**: A fully managed PaaS offering. Use **Single Database** for isolated workloads or **Elastic Pools** when multiple databases need to share a set of resources for cost efficiency.  
* **Azure SQL Managed Instance (MI)**: The "Hero" for migration. It is almost 100% compatible with on-premises SQL Server. **Crucial Fact**: Always link Managed Instance to requirements for **cross-database queries** and **SQL Agent** features.  
* **SQL on Azure VMs**: The "IaaS" option. Use this only when you need full OS-level access, specific SQL versions not available in PaaS, or third-party software that must run on the same server.

#### **💳 Billing Models and Tiers**

* **DTU vs. vCore**: Use **DTU** for simple, pre-configured bundles of compute and storage. Use **vCore** for granular control, allowing you to scale compute and storage independently.  
* **General Purpose**: Most common; balanced for standard workloads.  
* **Business Critical**: For high-performance apps with low-latency requirements and fast recovery using Always On replicas.  
* **Hyperscale**: For massive databases (up to 100TB) that require rapid scaling.  
* **Serverless**: Automatically scales compute based on workload demand and bills for compute used per second.

---

### 3.4 Migration Mechanisms

* **SSMA (SQL Server Migration Assistant)**: Used specifically for migrating non-SQL engines (like Oracle or MySQL) to Azure SQL.  
* **Azure SQL Migration Extension**: A specialized tool within Azure Data Studio for simplified, guided migrations.

---

### 3.5 Azure Cosmos DB

For "Multi-Active" scenarios requiring low read/write latency globally, Cosmos DB is the architect's choice.

* **APIs**: Know that you can migrate from MongoDB or MySQL using specific Cosmos APIs.  
* **Consistency Levels**: For the exam, focus on the trade-offs between **Strong** (guaranteed data order but higher latency) and **Bounded Staleness** (reads may lag slightly behind writes but offer better performance).

---

## **🌩️ Volume IV: Business Continuity and Disaster Recovery (BCDR)**

In the AZ-305 exam, success in the BCDR domain is measured by an architect's ability to meet strict Recovery Point Objective (RPO) and Recovery Time Objective (RTO) requirements. These metrics are abundant in exam scenarios and dictate which services you must deploy to ensure data integrity and service availability during a disaster.

---

### **4.1 Defining the Architectural Recovery Goals**

Before selecting a tool, you must understand the business constraints:

* **RPO (Recovery Point Objective)**: The maximum acceptable amount of data loss measured in time (e.g., "We can lose 1 hour of data").  
* **RTO (Recovery Time Objective)**: The maximum acceptable downtime for a service (e.g., "The site must be back up within 30 minutes").

---

### **4.2 Backup and Site Recovery Mechanisms**

Azure provides distinct tools for backup (data protection) and site recovery (business continuity).

#### **🔄 Azure Site Recovery (ASR)**

ASR is the primary tool for Business Continuity.

* **Orchestration**: It orchestrates the replication of virtual machines from a primary site to a secondary location.  
* **Failover and Failback**: It enables automated failover during a disaster and simplified failback once the primary site is healthy.  
* **Migration Use Case**: While designed for DR, ASR is frequently leveraged for migrating on-premises VMs to Azure.

#### **🛡️ Azure Backup**

Azure Backup focuses on long-term data retention and protection.

* **Ransomware Protection**: It protects against accidental deletion or targeted ransomware attacks by keeping data in isolated vaults.  
* **DR Objective**: While it can be part of a DR plan, it is usually for less demanding RPO/RTO requirements compared to ASR.  
* **Vault Selection**: Architects must distinguish between **Recovery Services Vault** and **Backup Vault**.  
  * **Recovery Services Vault**: Supports a wide range of data sources, including Azure VMs, SQL in Azure VMs, and SAP HANA.  
  * **Backup Vault**: Used for newer workloads like Azure Blobs, Azure Disks, and Azure Database for PostgreSQL.

---

### **4.3 Database High Availability and DR Options**

Expect approximately 3-5 questions focused on SQL-specific resiliency. You must know the differences and specific use cases for each.

#### **🚀 Always On Availability Groups (AGs)**

* **High Availability**: Provides the highest level of HA for SQL Server running on Azure Virtual Machines.  
* **Infrastructure**: Requires a Windows Server Failover Cluster (WSFC) and multiple VM instances.

#### **🔗 Failover Groups**

* **Managed Failover**: Provides a managed abstraction for Azure SQL Database and Azure SQL Managed Instance.  
* **Global Endpoint**: Uses a single listener endpoint that automatically points to the current primary region, simplifying application connection strings.

#### **🌍 Geo-Replication and Geo-Restore**

* **Geo-Replication**: Creates readable secondary databases in different regions. This is ideal for lower RPO requirements.  
* **Geo-Restore**: The most cost-effective but slowest recovery option, relying on geo-redundant backups. This is only suitable for very high RTO requirements.

---

### **4.4 Hybrid BCDR with Azure File Sync**

Azure File Sync acts as a bridge between on-premises and the cloud, serving both a performance and recovery role.

* **Centralization**: It allows you to move on-premises file shares to the cloud, creating centralized SMB shares in Azure.  
* **Local Access (Data Cache)**: It maintains local access speeds by caching frequently used files on the on-premises server.  
* **Rapid Restore**: In the event of an on-premises server failure, you can install the File Sync agent on a new server and quickly rebuild the local cache from the Azure central share.

---

## **🌐 Volume V: Infrastructure, Networking, and Monitoring**

*Networking and monitoring form the connective tissue of any Azure architecture. While foundational networking is a heavy focus in AZ-104, the AZ-305 exam shifts the perspective toward high-level design choices—specifically, how to route global traffic, secure service-to-service communication, and centralize observability.*

---

### **5.1 The Global vs. Regional Traffic Matrix**

An architect must know exactly which load-balancing service to choose based on the application's scope and protocol.

#### **🌍 Global Traffic Managers**

* **Azure Front Door**: A modern, global Layer 7 (HTTP/HTTPS) entry point that uses the Microsoft global edge network.  
  * **Architecture**: It utilizes Anycast to route users to the nearest "Point of Presence" (PoP).  
  * **Protection**: It brings integrated Web Application Firewall (WAF) protection, Rate Limiting, and SSL offloading.  
  * **Best For**: Global web applications requiring high availability and fast content delivery.  
* **Azure Traffic Manager**: A DNS-based load balancer that operates at the global level.  
  * **Mechanism**: It uses DNS to direct client requests to the most appropriate service endpoint based on a routing method (e.g., performance, priority, or geographic).  
  * **Limitation**: Because it is DNS-based, it does not "see" the traffic and therefore does not provide WAF or SSL offloading.

#### **📍 Regional Load Balancers**

* **Azure Application Gateway**: A regional Layer 7 load balancer.  
  * **Features**: Supports URL-path-based routing, cookie-based session affinity, and integrated WAF.  
  * **Use Case**: Ideal for complex web applications within a single region.  
* **Azure Load Balancer**: A high-performance, ultra-low-latency Layer 4 (TCP/UDP) load balancer.  
  * **Function**: It distributes inbound flows that arrive at the load balancer's front end to back-end pool instances.  
  * **Security**: Unlike the Layer 7 options, it does not provide WAF or URL-based routing.

---

### **5.2 Secure Connectivity and Hybrid Design**

Connecting Virtual Networks (VNets) and bridging the gap to on-premises environments are "imperative" skills.

#### **🔒 Private Connectivity: Link vs. Endpoint**

* **Private Link**: Provides private connectivity from a VNet to an Azure PaaS service (e.g., SQL, Storage).  
  * **Mechanism**: It creates a Private Endpoint with a private IP address inside your VNet, ensuring traffic stays on the Microsoft backbone and never touches the public internet.  
* **Service Endpoints (Service Link)**: Extends your VNet private address space and identity to Azure services over a direct connection.  
  * **Difference**: Unlike Private Link, the service still has a public IP, but access is restricted to your VNet at the network layer.

#### **🌉 Hybrid Networking**

* **ExpressRoute**: A private, dedicated connection from your on-premises data center to Azure. It provides higher security, more reliability, and faster speeds than typical internet-based connections.  
* **VPN Site-to-Site**: An encrypted tunnel over the public internet connecting your on-premises network to your Azure VNet.  
* **VNet Peering**: Directly connects two VNets in the same or different regions over the Azure backbone.  
* **Hub-and-Spoke**: A common architectural pattern where a central "Hub" VNet manages shared services (like ExpressRoute or Firewalls) and connects to multiple "Spoke" VNets containing individual workloads.

---

### **5.3 Diagnostic Tools and Monitoring**

Observability is mandatory for a Senior CSA. You must know how to leverage the data gathered from your infrastructure.

#### **🛠️ Network Watcher**

This is the "Swiss Army Knife" for troubleshooting network issues in Azure.

* **IP Flow Verify**: Checks if a packet is allowed or denied to/from a virtual machine based on Network Security Group (NSG) rules.  
* **NSG Flow Logs**: Provides detailed information about ingress and egress IP traffic through an NSG.  
* **Traffic Analytics**: A cloud-based solution that provides visibility into user and application activity across your networks.

#### **📊 Log Analytics Workspaces**

The Log Analytics Workspace is the central hub for log visualization and complex querying.

* **Centralization**: It is the mandatory destination for most Azure service diagnostic logs and activity logs when you need long-term retention or advanced analysis.  
* **Azure Metrics vs. Activity Logs**: While Metrics provide real-time numerical data and Activity Logs record "who did what," Log Analytics is where you go to cross-reference and visualize this data at scale.

---

### **5.4 Specialized Compute Networking**

* **RDMA Feature (H-series)**: High-performance H-series instances support Remote Direct Memory Access (RDMA) over InfiniBand.  
  * **Advantage**: This allows nodes in a cluster to communicate with extremely low latency and high throughput, which is essential for High-Performance Computing (HPC) workloads.

## **🚀 Volume VI: Compute and Web Application Design**

*The final pillar of the AZ-305 exam focuses on hosting strategies and high-availability (HA) compute. As an architect, your role is to determine which compute service—Virtual Machines, Scale Sets, or App Services—best aligns with the workload's performance requirements and the organization's administrative capabilities.*

---

### **6.1 Compute Resiliency and Availability**

Designing for "zero downtime" requires a deep understanding of how Azure handles hardware and datacenter failures. You must choose the right availability option based on the required Service Level Agreement (SLA).

* **Availability Sets**: These protect your application against hardware failures within a single datacenter by distributing VM instances across multiple **Fault Domains** (physical racks) and **Update Domains** (logical groups for patching).  
* **Availability Zones**: This is a higher level of protection that shields your application from a full datacenter outage by placing VM instances in physically separate datacenters within the same region.  
* **Virtual Machine Scale Sets (VMSS)**: This is the primary mechanism for **Autoscale**. It allows you to create and manage a group of load-balanced VMs where the number of instances can automatically increase or decrease in response to demand or a defined schedule.

---

### **6.2 App Service Hosting and Plans**

For web-based workloads, Azure App Service (PaaS) is often the preferred architectural choice over traditional VMs due to its reduced management overhead.

#### **📊 App Service Plan Tiers**

Choosing the correct tier is imperative for cost and feature availability:

* **Free/Shared**: Best for small development or testing environments; these tiers do not support scaling or custom domains.  
* **Basic**: Designed for apps that have lower traffic requirements and don't need advanced auto-scaling.  
* **Standard**: The production-ready tier that supports auto-scaling, daily backups, and up to 5 deployment slots.  
* **Premium (v2/v3)**: Designed for high-scale production apps; offers enhanced performance, larger storage, and advanced networking features like VNet integration.

#### **🔄 Web Application Migration**

* **App Service Migration Assistant**: Architects should leverage this tool to assess and migrate on-premises web applications to Azure App Service. It identifies compatibility issues and provides a guided path for moving the application code to the cloud.

---

### **6.3 Instance Types and Specialized Workloads**

The "alphabet soup" of VM sizes is more than just naming—it defines the underlying hardware capabilities.

* **B-series**: Burstable; ideal for workloads that are typically idle but need occasional high-performance bursts.  
* **D-series**: General purpose; balanced CPU-to-memory ratio for enterprise-grade applications.  
* **E-series**: Memory-optimized; high memory-to-core ratio for in-memory databases.  
* **F-series**: Compute-optimized; high CPU-to-memory ratio for batch processing or analytics.  
* **H-series**: High-performance computing (HPC); specifically designed for complex scientific modeling.  
  * **The RDMA Advantage**: H-series instances feature **Remote Direct Memory Access (RDMA)** over InfiniBand, which allows nodes to communicate with sub-microsecond latency and massive throughput.  
* **N-series**: GPU-enabled; designed for compute-intensive and graphics-intensive workloads like AI training or video rendering.

---

### **6.4 The "Always On" vs. Failover Decision**

For mission-critical applications, you must design for both High Availability (within a region) and Disaster Recovery (across regions).

* **Always On Availability Groups (AGs)**: Use this when you need the highest level of HA for SQL Server on Virtual Machines, providing multiple readable and writable replicas.  
* **Failover Groups**: Use this for a managed disaster recovery experience for Azure SQL Database, allowing for a single endpoint to handle regional failover.

---

## **Exam Strategy & Final Checklist: The Architect's Last Stand**

The AZ-305 isn't just a test of technical knowledge; it is a test of your ability to make **comparative architectural decisions**. As a Senior CSA, you know that the "best" solution is often the one that balances cost, complexity, and availability. Use this final checklist to sharpen your decision-making logic before stepping into the exam room.

---

### **🏛️ The Identity & Governance Decision Tree**

* **Need Just-in-Time (JIT) access?** Choose **Privileged Identity Management (PIM)**.  
* **Need to automate onboarding/offboarding via HR systems?** Use **Lifecycle Workflows**.  
* **Need to certify that users still require their access?** Perform an **Access Review**.  
* **Dealing with Partners or Auditors?** Use **B2B Collaboration**.  
* **Syncing complex multi-forest AD?** **Entra Connect Sync** is your tool; use **Cloud Sync** for lightweight, disconnected forests.  
* **Which license for PIM?** You must have **Entra ID P2 (E5)**.

### **💾 The Storage & SQL "Choose This" Matrix**

* **Need cross-database queries or SQL Agent after migration?** The answer is **Azure SQL Managed Instance**.  
* **Need to migrate a non-SQL database (MongoDB) to a globally distributed NoSQL?** Use **Azure Cosmos DB** with the appropriate API.  
* **Requirement is for 100TB+ database?** Choose the **Hyperscale** tier.  
* **Lowest possible latency for a managed disk?** Select **Ultra Disk**.  
* **Need to protect against a regional disaster?** You must have **GRS**, **RA-GRS**, **GZRS**, or **RA-GZRS** replication.

### **🌐 The Networking & Messaging Logic**

* **Global Layer 7 (Web) with WAF?** **Azure Front Door** is the primary choice.  
* **Global DNS-based routing (Non-HTTP)?** Use **Traffic Manager**.  
* **Regional Layer 7 with WAF?** **Application Gateway**.  
* **Millions of events/sec for telemetry?** **Event Hubs**.  
* **Enterprise messaging with 1:Many (Pub/Sub)?** **Service Bus Topics**.  
* **Private access to a PaaS service (SQL/Storage) without the Internet?** **Private Link**.

### **🌩️ The BCDR & Monitoring Strategy**

* **RPO is near-zero for VMs?** Use **Azure Site Recovery (ASR)**.  
* **Need to protect files against ransomware with long retention?** **Azure Backup** in a **Recovery Services Vault**.  
* **Need to troubleshoot why a packet is blocked?** Use **IP Flow Verify** in **Network Watcher**.  
* **Need to see who deleted a resource?** Check the **Activity Logs**.  
* **Need to query logs across multiple services?** Centralize them in a **Log Analytics Workspace**.

---

### **🏁 Final Pre-Exam Checklist**

* \[ \] I can distinguish between **Managed Identity** (no credential management) and **Service Principals**.  
* \[ \] I know that **Cosmos DB** consistency levels for the exam are usually **Strong** vs. **Bounded Staleness**.  
* \[ \] I remember that **RDMA** is the key differentiator for **H-series** VMs.  
* \[ \] I understand that **Self-hosted Integration Runtime** is required for **ADF** to reach on-prem data.  
* \[ \] I can identify the **6 replication types** and how many copies each creates.
