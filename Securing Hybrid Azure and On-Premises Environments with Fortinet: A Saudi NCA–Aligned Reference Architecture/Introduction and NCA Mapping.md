
# Securing Hybrid Azure and On-Premises Environments with Fortinet: A Saudi NCA–Aligned Reference Architecture (0/6)

**Disclaimer:** The views and opinions expressed in this series are my own. For official reference architectures, product guidance, and implementation recommendations, please refer to the official Microsoft and Fortinet documentation and websites.

## Introduction

Microsoft announced that its Saudi Arabia East datacenter region is expected to be available to customers in November 2026. The region is intended to provide access to supported Microsoft cloud and AI services and to enable eligible workloads and data to be hosted locally in the Kingdom.

Saudi organizations are accelerating their use of cloud computing, artificial intelligence, Internet of Things, and other digital services. For many organizations, Microsoft Azure is becoming an extension of the existing technology estate rather than a replacement for it. Core business systems, legacy applications, operational-technology environments, branch networks, retail locations, and other workloads may remain on-premises while new applications and services are deployed in Azure.

The resulting environment is hybrid. The security challenge is therefore not simply to secure Azure. It is to maintain consistent security principles, visibility, governance, and operational processes across on-premises and cloud environments while still using cloud-native capabilities where they are most appropriate.

Without a deliberate architecture, security teams can end up operating different policy models, different logging platforms, separate control evidence, and disconnected incident-response processes. A network access policy implemented on an on-premises firewall, for example, may be represented very differently by Azure routing, network security groups, native platform controls, identity policies, and cloud security services. The hybrid architecture must make those relationships explicit.

I am writing a series or articles that introduces a logical reference architecture built around the Fortinet Security Fabric and selected cloud-delivered Fortinet services. It explains where each capability can support the hybrid design and selected National Cybersecurity Authority (NCA) controls. This reference architecture is intended to help Saudi organizations design and operate hybrid on-premises and Microsoft Azure environments with security capabilities that can support selected requirements in the **Essential Cybersecurity Controls (ECC-2:2024)** and the **Cloud Cybersecurity Controls (CCC-2:2024)**. It is an architecture guide, not a certification, legal opinion, or a determination of regulatory compliance.

ECC-2:2024 applies to Saudi government agencies, their affiliated companies and entities, and private-sector entities that own, operate, or host Critical National Infrastructure (CNI). For these entities, ECC requires ongoing and continuous compliance with applicable controls. NCA strongly encourages other entities in the Kingdom to use ECC as a cybersecurity best-practice baseline. ECC cloud-computing and hosting controls are applicable to entities that use or plan to use cloud computing and hosting services.

CCC-2:2024 extends ECC for cloud computing. It distinguishes between the **Cloud Service Provider (CSP)**, which provides a cloud service, and the **Cloud Service Tenant (CST)**, which consumes it. CCC applies to CSPs that provide cloud services to in-scope CSTs and to in-scope CSTs, including Saudi government agencies and private-sector entities that own, operate, or host CNI and use or plan to use cloud services. Other organizations may use CCC as a best-practice baseline.

In this first article of the series, we introduce the reference architecture at a glance, followed by a high-level overview of its key technical capabilities and selected areas of alignment with NCA requirements. Please keep in mind that the individual solutions, architectural components, and technical aspects will be explored in greater detail in the subsequent articles in the series.

## The reference architecture at a glance

![alt text](<Fortinet Azure Landing Zone Reference Architecture.png>)

The architecture is a logical reference design for a hybrid Azure and on-premises environment. It shows intended trust boundaries, security functions, and integration patterns; it is not a physical deployment specification or a complete NCA control implementation.

The design has four principal zones.

1. **Azure landing zone.** The Azure estate is organized into a platform foundation and application landing zones. Platform capabilities centralize identity integration, connectivity, management, governance, and security monitoring. Application landing zones separate workloads by business function, ownership, environment, and risk. This follows the Azure landing-zone principle that platform and workload environments should be governed through a common management structure and reusable guardrails.
2. **On-premises environment.** A FortiGate SD-WAN hub at the corporate data center connects branch FortiGates and reaches Azure through redundant ExpressRoute private peering.
3. **Remote users and sites.** Managed user devices, home workers, retail and thin-edge sites, and authorized third-party partners connect through a defined secure-access design. The pattern uses identity, device posture, least privilege, and segmented access rather than broad network reachability.
4. **Cloud-delivered security services.** FortiSASE can provide secure web gateway, ZTNA, firewall-as-a-service, and CASB capabilities for designated users and sites. FortiCNAPP can provide posture, workload, identity-entitlement, and code-security capabilities across Azure and the DevOps pipeline. FortiAppSec Cloud can provide cloud-delivered web-application and API protection where that deployment pattern is selected.

## Technical capabilities and selected NCA alignment

The following sections map selected technology capabilities to relevant ECC-2:2024 and CCC-2:2024 controls. For CCC references, T denotes a Cloud Service Tenant control as this article focuses primarily on tenant-operated controls. The abbreviated control descriptions below are derived from ECC-2:2024 and CCC-2:2024.

### Network security and hybrid connectivity

The hybrid network foundation enforces approved connectivity, segmentation, inspection, encryption, and telemetry patterns across Azure, on-premises systems, branches, remote sites, and third-party connections.
#### FortiGate

FortiGate-VM is a next-generation firewall delivered as a virtual appliance. In this reference design, it can be deployed in the connectivity hub VNet and, where required, alongside Azure VMware Solution. It provides the policy-enforcement point for designated north-south, east-west, and hybrid flows. The design must use explicit route control, policy zones, high-availability placement, and validated traffic-steering patterns to ensure that the intended flows actually traverse the inspection point.

Using FortiOS across on-premises and Azure can support a common policy lifecycle and reusable policy objects. It does not remove the need to design Azure routing, native access controls, platform governance, and cloud-specific management separately.

| Framework | Control   | Abbreviated requirement                                    | How the capability can support the control                                                                                                  |
| --------- | --------- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| ECC       | 2-5-3-1   | Secure segmentation using firewall and defense in depth    | Hub-and-spoke inspection and zone policies can enforce approved segmentation between spokes and between Azure and on-premises environments. |
| ECC       | 2-5-3-2   | Isolation of production from test and development networks | Separate subscriptions, spokes, route domains, and policy zones can implement the approved environment-separation model.                    |
| ECC       | 2-5-3-5   | Restrict and manage services, protocols, and ports         | Application-aware policies can restrict permitted services and ports.                                                                       |
| ECC       | 2-5-3-6   | Intrusion Prevention Systems                               | IPS can be applied to defined inspected traffic flows.                                                                                      |
| ECC       | 2-8-3-3   | Encrypt data in transit and at rest as applicable          | IPsec, and encrypted transport patterns can support in-transit requirements; the data and key-management design remains necessary.          |
| CCC       | 2-4-T-1-1 | **CST:** protect the connection channel with the CSP       | The selected encrypted and inspected hybrid channel can support the tenant control.                                                         |
| CCC       | 2-7-T-1-1 | **CST:** strong cryptography aligned to NCS-1:2020         | Supported cryptographic settings can be selected in accordance with the entity’s classification, risk assessment, and applicable level.     |
| CCC       | 2-7-T-1-2 | **CST:** encrypt data transferred to or from the cloud     | An approved IPsec overlay, MACsec where applicable, and TLS-enabled application paths can support this requirement.                         |
#### FortiProxy

FortiProxy can provide secure-web-gateway capabilities for defined Azure-hosted users and server egress paths, such as Azure Virtual Desktop sessions. The design must deliberately steer applicable web traffic to the proxy and define exclusions, certificate handling, privacy limits, and availability behavior.

| Framework | Control | Abbreviated requirement                   | How the capability can support the control                                                               |
| --------- | ------- | ----------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| ECC       | 2-5-3-3 | Secure browsing and internet connectivity | Category-based URL filtering and application control can restrict defined high-risk browsing categories. |
| ECC       | 2-5-3-8 | Protect the browsing channel against APTs | TLS inspection and sandbox integration can support inspection of eligible web traffic.                   |
| ECC       | 2-7-2   | Protect data according to classification  | DLP policies on eligible outbound web traffic can support approved data-handling rules.                  |
#### FortiNDR

FortiNDR can analyze network traffic to detect attacker behavior such as lateral movement, data exfiltration, and privilege escalation. Its coverage depends on the telemetry design. It can observe traffic that is delivered to its sensors, mirrored through the selected Azure telemetry pattern, or otherwise integrated with the architecture. It does not automatically observe all intra-VNet, PaaS, private-endpoint, or encrypted traffic.

| Framework | Control | Abbreviated requirement | How the capability can support the control |
| --- | --- | --- | --- |
| ECC | 2-12-3-4 | Continuous monitoring of cybersecurity event logs | Network detections can be forwarded to the SOC for continuous monitoring and investigation. |
| ECC | 2-13-3 | Cybersecurity incident and threat management | NDR detections can provide an investigation and response input. |

### Identity and privileged access

Identity remains a cross-cutting architecture service. Microsoft Entra ID should remain the authority for the Azure control plane and Microsoft 365, while Fortinet components can integrate with the selected identity architecture for network access, ZTNA, privileged sessions, and security-device administration. The design must distinguish workload and infrastructure administration from Azure control-plane privilege.
#### FortiAuthenticator

FortiAuthenticator can provide multi-factor authentication with FortiToken, single sign-on, user-identity mapping for identity-aware firewall policy, and certificate services within the selected design. It can federate with Microsoft Entra ID. The exact authentication authority, conditional-access decision, directory source, certificate lifecycle, and fallback behavior must be defined before deployment.

| Framework | Control   | Abbreviated requirement                                  | How the capability can support the control                                                                |
| --------- | --------- | -------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| ECC       | 2-2-3-1   | Single-factor authentication using username and password | Centralized authentication against approved directory sources can support baseline authentication.        |
| ECC       | 2-2-3-2   | MFA for remote access and privileged accounts            | MFA can be applied to defined VPN, ZTNA, and security-device administrative paths.                        |
| ECC       | 2-2-3-3   | Need-to-know, least privilege, and segregation of duties | Group and role data can drive identity-aware policy.                                                      |
| ECC       | 2-4-3-2   | MFA for remote and webmail access                        | The selected identity-control design can enforce MFA for applicable access paths.                         |
| ECC       | 2-15-3-5  | Authentication for web applications                      | Authentication services can support published-application access patterns.                                |
| CCC       | 2-2-T-1-4 | **CST:** MFA for privileged cloud users                  | MFA can support privileged-access paths into the Azure estate.                                            |
| CCC       | 2-2-T-1-5 | **CST:** detect and prevent unauthorized cloud access    | Lockout policies, authentication telemetry, and response procedures can support detection and prevention. |
#### FortiPAM

FortiPAM can provide credential vaulting, password rotation, brokered access, and session recording for defined privileged sessions into Azure workloads and on-premises systems. Azure control-plane privilege should still be governed through the selected Entra ID, Azure RBAC, and privileged-access model. FortiPAM should be positioned as the broker for defined workload, operating-system, application, network-device, and infrastructure administration sessions.

| Framework | Control   | Abbreviated requirement                                | How the capability can support the control                                                     |
| --------- | --------- | ------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| ECC       | 2-2-3-4   | Privileged access management                           | Vaulted credentials and brokered privileged sessions can support the approved PAM process.     |
| ECC       | 2-12-3-2  | Logs for privileged accounts and remote access         | Session recording and audit logs can contribute to required evidence.                          |
| CCC       | 2-2-T-1-1 | **CST:** manage cloud credentials over their lifecycle | Credential rotation and lifecycle controls can support defined cloud and workload credentials. |
| CCC       | 2-2-T-1-3 | **CST:** secure cloud session management               | Session controls, termination, timeout, and audit logging can support the tenant requirement.  |

### Application, API, and AI security

Application-security components should be selected according to the application’s exposure, protocol, API requirements, deployment model, global traffic needs, latency constraints, data-handling requirements, and operational owner. The architecture should not imply that FortiWeb, FortiAppSec Cloud, FortiADC, and FortiAIGate must all be in every application path.
#### FortiWeb

FortiWeb can provide web-application firewall protection for internet-facing applications and APIs hosted in Azure. It may be deployed in an in-VNet application-security pattern or in front of specified application spokes. The production design must show the TLS, certificate, load-balancing, health-check, DDoS, logging, and failure-mode paths.

| Framework | Control  | Abbreviated requirement            | How the capability can support the control                                                                                   |
| --------- | -------- | ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| ECC       | 2-15-3-1 | Use a web application firewall     | WAF policies can protect designated published applications and APIs.                                                         |
| ECC       | 2-15-3-2 | Adopt a multi-tier architecture    | Reverse-proxy placement can support separation between presentation and backend tiers when combined with the network design. |
| ECC       | 2-15-3-3 | Use secure protocols such as HTTPS | TLS termination and enforcement can support the secure-protocol requirement.                                                 |
| ECC       | 2-5-3-9  | Protect against DDoS attacks       | Application-layer mitigation can support the approved DDoS protection strategy.                                              |
#### FortiAppSec Cloud

FortiAppSec Cloud can provide SaaS-delivered web-application and API protection, including WAF, API security, bot protection, DDoS mitigation, and global server load balancing, where a cloud-delivered edge pattern is selected.

| Framework | Control | Abbreviated requirement | How the capability can support the control |
| --- | --- | --- | --- |
| ECC | 2-15-3-1 | Use a web application firewall | Cloud-delivered WAF policies can protect designated web applications and APIs. |
| ECC | 2-15-3-3 | Use secure protocols | TLS enforcement at the service edge can support secure-protocol requirements. |
| ECC | 2-5-3-9 | Protect against DDoS attacks | Cloud-delivered scrubbing and mitigation capabilities can support the DDoS strategy. |
| ECC | 4-1 and 4-2 | Third-party and cloud-computing cybersecurity | The service can be assessed within the third-party and cloud-service scope, but does not itself satisfy contractual, risk, data-return, or provider-separation obligations. |
#### FortiADC

FortiADC can provide load balancing, TLS offload, global server load balancing, and application delivery capabilities for selected application and AI workloads. It can help manage traffic between client-facing services and backend application components, including workloads hosted in Kubernetes, while supporting application availability and resilient service delivery.

| Framework | Control | Abbreviated requirement | How the capability can support the control |
| --- | --- | --- | --- |
| ECC | 2-15-3-2 | Adopt a multi-tier architecture | Reverse-proxy and load-balancing patterns can support separation of the client-facing tier from backend services. |
| ECC | 2-15-3-3 | Use secure protocols | TLS termination and re-encryption can support approved secure-protocol patterns. |
| ECC | 2-8-3-3 | Encrypt data in transit and at rest as applicable | Encrypted client and server-side sessions can support in-transit protection. |
| ECC | 3-1-3 | Include cybersecurity resilience in business continuity | Load balancing and GSLB can support the availability design as one element of continuity planning. |
#### FortiAIGate

FortiAIGate can be placed on the application-to-model path for defined AI use cases. It can inspect prompts and responses for prompt injection, sensitive-data exposure, model abuse, and excessive consumption, and can produce telemetry for monitoring and investigation. It should be paired with an approved AI-use policy, data-classification rules, prompt and model governance, and a clear definition of whether AI interactions may be retained or exported.

| Framework | Control | Abbreviated requirement | How the capability can support the control |
| --- | --- | --- | --- |
| ECC | 2-7-2 | Protect data according to classification | Prompt and response inspection can support approved controls that prevent sensitive data from reaching or leaving a model. |
| ECC | 2-15-3 | Protect external web applications | The gateway can support the application-security design where the AI application is externally exposed. |
| ECC | 2-12-3-1 | Activate logs for critical information assets | AI interaction telemetry can contribute to monitoring and investigation evidence. |
### Threat protection: email, files, malware, and cloud-native workloads

#### FortiSandbox

FortiSandbox can perform dynamic analysis of unknown files in an isolated environment. In this design, it can receive content from integrated FortiMail, FortiGate, and FortiProxy controls and can scan eligible content in Microsoft 365 SharePoint and OneDrive and Azure storage. The implementation must define connector permissions, content scope, quarantine location, residency, file-size and type limits, scan latency, failure handling, and exceptions.

| Framework | Control | Abbreviated requirement | How the capability can support the control |
| --- | --- | --- | --- |
| ECC | 2-3-3-1 | Protect systems from malware using advanced technologies | Behavioral analysis of eligible unknown files can support advanced malware protection. |
| ECC | 2-4-3-4 | Protect email against APTs | Detonation of eligible email attachments and URLs can support email protection. |
| ECC | 2-5-3-8 | Protect the browsing channel against APTs | Analysis of eligible downloaded files can support the browsing-protection design. |
#### FortiMail

FortiMail can provide secure-email-gateway capabilities for Exchange Online, including phishing, business-email-compromise, impersonation, spam, and malware protections. The mail-flow design must define inbound and outbound routing, exception handling, service availability, archive scope, tenant-domain configuration, and responsibilities for DNS records.

| Framework | Control | Abbreviated requirement                     | How the capability can support the control                                               |
| --------- | ------- | ------------------------------------------- | ---------------------------------------------------------------------------------------- |
| ECC       | 2-4-3-1 | Analyze and filter phishing and spam emails | Multi-layer inbound filtering can support the approved email-protection design.          |
| ECC       | 2-4-3-3 | Email archiving and backup                  | Archive capability can support the entity’s approved archiving and backup requirements.  |
| ECC       | 2-4-3-4 | Protect against APTs                        | Integration with FortiSandbox can support analysis of eligible content.                  |
| ECC       | 2-4-3-5 | Validate domains using SPF, DKIM, and DMARC | Inbound validation and outbound DKIM signing can support domain-protection requirements. |
#### FortiCNAPP

FortiCNAPP can provide cloud-native application-protection capabilities across posture management, identity entitlements, workload and Kubernetes protection, and code security. In this architecture, it can connect to defined Azure Resource Manager, Entra ID, Kubernetes, and DevOps sources. The entity must approve read and write permissions, define remediation workflows, validate the inventory scope, and integrate findings into risk, change, and vulnerability-management processes.

| Framework | Control              | Abbreviated requirement                                                                 | How the capability can support the control                                                            |
| --------- | -------------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| ECC       | 1-3-3                | Support policies with technical security standards                                      | Posture assessments can identify configuration drift against approved baselines.                      |
| ECC       | 1-6-3-1              | Use secure coding standards                                                             | Static analysis can support assessment of first-party code.                                           |
| ECC       | 1-6-3-2              | Use trusted development tools and libraries                                             | Software-composition analysis and SBOM capabilities can support library governance.                   |
| ECC       | 1-6-3-3              | Test software for compliance                                                            | Code and infrastructure-as-code scanning can support pre-deployment assessment.                       |
| ECC       | 2-2-3-3              | Apply least privilege                                                                   | CIEM analysis can identify effective permissions and potential excess access.                         |
| ECC       | 2-2-3-5              | Review identities and access rights periodically                                        | Continuous identity-risk reporting can support periodic access reviews.                               |
| ECC       | 2-10-3-1 to 2-10-3-3 | Assess, classify, and remediate vulnerabilities                                         | Workload and container scanning can support vulnerability discovery and prioritization.               |
| CCC       | 2-1-T-1-1            | **CST:** inventory cloud services and related assets                                    | Continuous discovery can support the cloud-asset inventory.                                           |
| CCC       | 2-2-T-1-1            | **CST:** manage cloud credentials over their lifecycle                                  | CIEM visibility can support lifecycle oversight of Entra and Azure RBAC permissions.                  |
| CCC       | 2-9-T-1-1            | **CST:** assess and remediate cloud-service vulnerabilities at least every three months | Continuous scanning can support the required assessment frequency, subject to remediation governance. |
### Secure access for users and sites

#### FortiSASE

FortiSASE can provide cloud-delivered secure web gateway, firewall-as-a-service, ZTNA, and CASB capabilities for defined remote users, thin-edge sites, and third-party partner connections. The architecture must identify the selected regional service, traffic-steering method, identity provider, device-posture requirements, inspection policy, log location and retention, support-access model, and the data-handling and provider-risk assessment.

| Framework | Control   | Abbreviated requirement                              | How the capability can support the control                                                                |
| --------- | --------- | ---------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| ECC       | 2-2-3-2   | MFA for remote access                                | ZTNA integrated with the selected identity provider can enforce MFA for defined remote paths.             |
| ECC       | 2-5-3-3   | Secure browsing and internet connectivity            | Cloud-delivered URL filtering and secure-web-gateway capabilities can support approved browsing controls. |
| ECC       | 2-5-3-8   | Protect the browsing channel against APTs            | TLS inspection and sandbox integration can support inspection of eligible traffic.                        |
| ECC       | 2-6-3-2   | Control mobile-device use based on business need     | Device-posture conditions can support controlled access from managed devices.                             |
| ECC       | 2-12-3-2  | Log remote access                                    | Centralized remote-session logging can contribute to evidence.                                            |
| CCC       | 2-4-T-1-1 | **CST:** protect the connection channel with the CSP | Secure private access patterns can support protected access into Azure.                                   |
#### FortiClient

FortiClient is an endpoint agent that can provide ZTNA and VPN connectivity to FortiSASE, device posture for access decisions, endpoint protection, vulnerability scanning, and Security Fabric telemetry, depending on the edition and selected configuration. The entity must define the device-management boundary, operating-system coverage, EPP versus EMS capabilities, patching responsibilities, privacy rules, and response procedures.

| Framework | Control  | Abbreviated requirement                          | How the capability can support the control                                                                            |
| --------- | -------- | ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| ECC       | 2-3-3-1  | Protect workstations and servers from malware    | Endpoint protection and sandbox integration can support malware protection where the appropriate edition is deployed. |
| ECC       | 2-3-3-2  | Restrict external storage media                  | USB device control can support approved removable-media restrictions.                                                 |
| ECC       | 2-3-3-3  | Patch systems, applications, and devices         | Vulnerability visibility and patching integrations can support the patch-management process.                          |
| ECC       | 2-6-3-2  | Control mobile-device use based on business need | Device posture can be checked before access is granted.                                                               |
| ECC       | 2-10-3-1 | Perform periodic vulnerability assessment        | Endpoint vulnerability visibility can contribute to assessment coverage.                                              |

### Management, visibility, and security operations

#### FortiManager

FortiManager can centralize policy and configuration management for supported FortiGate deployments across Azure, on-premises, branches, and Azure VMware Solution. 

A common policy lifecycle can reduce operational variation, but it does not make cloud-native routing, Azure Policy, Entra configuration, native logs, or application-service controls identical to FortiGate policy. The entity must define which policies are authoritative in each control plane and how conflicts are reviewed.

| Framework | Control | Abbreviated requirement                                 | How the capability can support the control                                   |
| --------- | ------- | ------------------------------------------------------- | ---------------------------------------------------------------------------- |
| ECC       | 1-3-3   | Support policies with technical security standards      | Templates and policy packages can support standardized configuration.        |
| ECC       | 1-6-1   | Include cybersecurity requirements in change management | Change workflows and approvals can support policy-change governance.         |
| ECC       | 1-6-2-2 | Review secure configuration before changes go live      | Revision history and pre-install review can support configuration assurance. |
| ECC       | 2-5-4   | Review network-security requirements periodically       | Central policy visibility can support periodic rule and requirement reviews. |
#### FortiAnalyzer

FortiAnalyzer can collect, analyze, and retain telemetry from integrated Fortinet devices. It can support a central Fortinet logging architecture, analytics, automation, and threat-intelligence workflows. It should not be described as a substitute for Azure platform, Entra, Microsoft 365, application, database, or other cloud logs that must be collected through their own diagnostic settings and connectors.

| Framework | Control | Abbreviated requirement | How the capability can support the control |
| --- | --- | --- | --- |
| ECC | 2-12-3-1 | Activate logs for critical information assets | Central collection from integrated Fortinet devices can support the logging design. |
| ECC | 2-12-3-2 | Log privileged-account and remote-access events | Administrative and VPN/ZTNA logs can be retained centrally. |
| ECC | 2-12-3-4 | Monitor cybersecurity event logs continuously | Dashboards and detections can support continuous monitoring. |
| ECC | 2-12-3-5 | Retain logs for at least 12 months | A sized, protected retention design can support the retention requirement. |
| CCC | 2-11-T-1-1 | **CST:** activate and collect cloud login and cybersecurity logs | Logs from Azure-hosted Fortinet components can contribute to tenant evidence. |
| CCC | 2-11-T-1-2 | **CST:** monitor activated cloud cybersecurity logs | A unified monitoring view can support monitoring of the integrated sources. |
#### FortiSIEM

FortiSIEM can correlate events across Fortinet and non-Fortinet sources, including Azure activity logs, Entra sign-in logs, and Microsoft 365, and can maintain a CMDB based on discovered assets. The collection design must specify every required source, connector, diagnostic setting, ingestion latency, retention period, integrity protection, time synchronization, access control, monitoring rule, and escalation path.

CMDB attributes can support asset-classification tracking. They do not, by themselves, establish data classification, asset ownership, or handling requirements; those remain governed by the entity’s asset-management process. 

| Framework | Control | Abbreviated requirement | How the capability can support the control |
| --- | --- | --- | --- |
| ECC | 2-1-5 | Classify, label, and handle assets | CMDB attributes can support tracking of the approved asset-classification process. |
| ECC | 2-12-3-3 | Identify required SIEM techniques | Correlation rules and UEBA can support the selected SIEM approach. |
| ECC | 2-12-3-4 | Monitor cybersecurity event logs continuously | Correlation and alerting can support continuous monitoring. |
| CCC | 2-1-T-1-1 | **CST:** inventory cloud services and related assets | Automated discovery can contribute to the cloud-asset inventory. |
| CCC | 2-11-T-1-1 | **CST:** activate and collect cloud login and cybersecurity logs | Azure and Entra log ingestion can support tenant log collection. |
| CCC | 2-11-T-1-2 | **CST:** monitor activated cloud cybersecurity logs | Correlation across cloud and on-premises sources can support monitoring coverage. |
#### FortiSOAR

FortiSOAR can provide playbook-driven response, case management, and threat-intelligence handling. It may include AI-assisted functions, but automated and AI-assisted actions must be governed by approved incident-response procedures, analyst authorization, escalation thresholds, change controls, and evidence requirements. AI assistance is not a substitute for incident plans, NCA reporting, exercises, chain of custody, or management accountability.

| Framework | Control | Abbreviated requirement | How the capability can support the control |
| --- | --- | --- | --- |
| ECC | 2-13-3-1 | Maintain incident-response plans and escalation procedures | Approved response plans can be implemented as reviewable playbooks. |
| ECC | 2-13-3-2 | Classify incidents | Case-management fields and workflows can support incident classification. |
| ECC | 2-13-3-5 | Collect and handle threat-intelligence feeds | Threat-intelligence management and enrichment can support defined handling processes. |

## What comes next

The following articles should deepen each layer of the architecture through design patterns, traffic flows, integration points, high-availability decisions, operating procedures, and evidence requirements.

1. The Azure landing zone and FortiGate hub. Segmentation, routing, inspection patterns, private endpoints, DNS, Azure-native control-plane protections, ExpressRoute, SD-WAN, BGP, redundant customer-edge design, encryption options, key management, and traffic validation between on-premises and Azure.
2. Identity-aware security. Entra ID, Azure RBAC and privileged access, FortiAuthenticator, FortiPAM, identity-based policy, access reviews, and emergency access.
3. Application, API, and AI security. Selection patterns for FortiWeb, FortiAppSec Cloud, FortiADC, and FortiAIGate; ingress, TLS, WAF, API, DDoS, and AI runtime controls.
4. Cloud-native protection from code to runtime. FortiCNAPP, workload inventory, posture management, CIEM, code and infrastructure-as-code scanning, remediation, and evidence.
5. Secure access for users, sites, and partners. FortiSASE, FortiClient, ZTNA, device posture, thin-edge sites, third-party access, logging, and provider-risk considerations.
6. Security operations across the hybrid estate. FortiManager, FortiAnalyzer, FortiSIEM, FortiSOAR, Azure and Entra diagnostics, source coverage, retention, detection, response, and playbook governance.

The objective of the series is to help cloud and security teams accelerate Azure adoption while incorporating cybersecurity requirements from the beginning. The strongest implementation will use Fortinet and Microsoft capabilities together, within a documented shared-responsibility model and an evidence-based NCA compliance program, rather than attempting to retrofit governance after the cloud environment has been built.


References

[Microsoft Announces That Saudi Arabia East Datacenter Region Will Be Available in November 2026](https://news.microsoft.com/source/emea/2026/08/microsoft-announces-saudi-arabia-east-datacenter-region-will-be-available-in-november-2026/)
[Essential Cybersecurity Controls, ECC-2:2024](https://cdn.nca.gov.sa/api/files/public/upload/86e09090-44e4-481f-bc28-355673607654_ECC--2024-EN.pdf)
[Cloud Cybersecurity Controls, CCC-2:2024](https://cdn.nca.gov.sa/api/files/public/upload/6d5408a3-d8e6-4e96-963b-2c7198e5b7c2_CCC-2-2024-EN-.pdf)
[Mapping Fortinet Security Fabric with NCA ECC](https://www.fortinet.com/content/dam/maindam/PUBLIC/02_MARKETING/02_Collateral/SolutionBrief/sb-NCA-ECC-KSA-2025-01-a4.pdf)
