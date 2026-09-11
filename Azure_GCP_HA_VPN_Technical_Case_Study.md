# Technical Case Study: Secure Azure-to-GCP Integration for Real-Time Payment Fraud Decisions

## Case Study Purpose

This technical case study extends the original Azure-GCP HA VPN proof of concept into a hypothetical production architecture for a regulated financial institution.

The original project demonstrates resilient Azure-to-GCP connectivity using Azure VPN Gateway, Google Cloud HA VPN, redundant IPsec/IKEv2 tunnels, BGP dynamic routing, and cross-cloud network controls. This case study uses that technical foundation to examine the additional security, resilience, identity, data, monitoring, and operational decisions I would evaluate before using cross-cloud connectivity for a real-time payment workload.

This is an architecture case study, not a claim that the full banking scenario described below was implemented in the original PoC.

## Business and Application Scenario

For this scenario, a financial institution operates a customer-facing digital banking and real-time payments application in Microsoft Azure. The application supports payment initiation and must obtain a fraud/risk decision before certain payments are released to a real-time payment rail.

The organization already has an enterprise fraud and risk analytics capability in Google Cloud Platform (GCP). The multicloud decision has therefore already been made. The architecture problem is how to integrate the Azure payment environment with the GCP fraud capability securely, reliably, and without turning the cross-cloud connection into broad network trust.

The GCP fraud service is assumed to return a risk result such as ALLOW, CHALLENGE, HOLD, or DENY. The payment application applies the institution's approved payment and fraud policy to that result.

## Scope and Assumptions

### Fraud Platform Scope

The fraud analytics capability and its fraud-detection logic are treated as existing enterprise services. This case study does not design the fraud model or determine how individual fraud signals are calculated.

For the scenario, I assume the fraud platform requires a limited set of approved transaction and contextual signals, such as payment amount, tokenized customer/account identifiers, recipient information, transaction velocity, authentication risk, and device/session risk.

The architecture focuses on securely and resiliently delivering those approved signals to the GCP fraud capability and receiving its decision.

### Fraud Decision Timeout Assumption

For this case study, the payment workflow allows up to **30 seconds** for the GCP fraud service to return a decision.

The 30-second threshold is a hypothetical design assumption used to demonstrate timeout handling, failure behavior, and business-continuity architecture. It should not be interpreted as a FedNow, RTP, or industry-standard processing requirement.

If the fraud decision is not received within the assumed window, the payment does not remain indefinitely in an uncertain state. The payment orchestration invokes the institution's predefined fraud-continuity policy.

### Production Connectivity Assumption

The original PoC demonstrates redundant VPN connectivity. In a production financial-services environment, I would also evaluate whether availability, latency, transaction volume, recovery objectives, regulatory requirements, and cost justify dedicated private connectivity as a primary path with encrypted VPN connectivity as an independent backup.

That is a production architecture recommendation, not functionality implemented in the original PoC.

## Architecture Overview

I would separate the integration into two distinct flows because they have different timing and recovery requirements.

### Real-Time Fraud Decision Path

The synchronous path supports a payment that is waiting for a fraud decision:

**Customer -> Azure Digital Banking -> Azure Payment Service -> Secure Azure/GCP Connection -> GCP Fraud Service -> Decision Returned -> Azure Payment Service -> Real-Time Payment Rail**

The payment service sends only the approved fraud attributes needed for the decision. A unique transaction/correlation identifier follows the request and response so the interaction can be traced without using customer PII as the primary log identifier.

A representative request could contain:

`Transaction ID | Tokenized Customer/Account Reference | Payment Attributes | Approved Risk Signals | Timestamp`

A representative response could contain:

`Transaction ID | Risk Decision | Risk Score/Category | Response Timestamp`

The fraud response must arrive within the institution's defined processing window. For this case study, that window is assumed to be 30 seconds.

### Asynchronous Analytics and Recovery Path

Not every event needs to participate in the synchronous payment decision. Payment, security, and operational events can also be delivered asynchronously for broader fraud analysis, historical analytics, model support, and operational use.

For that path:

**Azure Payment/Event Service -> Azure Durable Messaging -> GCP Analytics Consumer**

I would keep the durable messaging layer on the Azure side because Azure is the system producing the events. GCP availability should not determine whether Azure can preserve an event.

A service such as Azure Service Bus is a reasonable implementation option for this scenario. If GCP or the cross-cloud consumer becomes unavailable, events remain retained in Azure and processing resumes after recovery.

The real-time and asynchronous paths therefore use different failure strategies:

- Real-time path failure: bounded timeout and predefined business fallback policy.
- Asynchronous path failure: durable retention and backlog recovery.

## Data Protection and Minimization

Before establishing connectivity, I would identify exactly what data the fraud service requires.

The fact that a secure cross-cloud connection exists does not justify moving additional banking data into GCP. Names, full account numbers, addresses, credentials, Social Security numbers, or other direct identifiers should not be transferred unless the fraud use case and organizational policy require them.

Where analytics can operate on tokenized customer or account references, I would prefer tokenization over unnecessary exposure of direct identifiers.

The data set should have:

- defined ownership;
- data classification;
- approved business purpose;
- minimum required fields;
- retention requirements;
- access requirements; and
- audit requirements.

Changes to the data set or its intended use should trigger review rather than being treated as a routine interface change.

## Cross-Cloud Network Topology

The VPN is a protected transport path, not a declaration that Azure and GCP are one trusted network.

A conceptual topology is:

**Azure Payment/Application Zone -> Azure Routing and Security Boundary -> Azure VPN Gateway -> Redundant Encrypted Tunnels -> GCP HA VPN / Cloud Router -> GCP Fraud Processing Zone**

The original PoC's redundant VPN and BGP architecture provides the technical foundation for this design.

In production, I would separate payment/application, integration/messaging, fraud processing, analytics, and management functions into appropriate network/security zones. Cross-cloud access would be explicitly permitted according to workload and business purpose.

## Routing and BGP

BGP provides dynamic route exchange and supports failover between redundant paths, but I would not advertise entire Azure and GCP address spaces simply because the clouds are connected.

Only the prefixes required for the fraud integration should be exchanged.

Azure should advertise only the networks that approved GCP workloads need to reach. GCP should advertise only the networks required for the Azure payment environment to reach the fraud service.

The principle is:

**A route determines where traffic can go. It does not determine whether the traffic is authorized.**

Authorization still depends on network security policy and application/workload identity.

During failure testing, route withdrawal and reconvergence should be validated to ensure that loss of a tunnel moves traffic to the intended healthy path without exposing unintended networks.

## Segmentation and Firewall Policy

Cross-cloud traffic should be denied by default.

The primary approved flows are:

1. Azure Payment Service to the GCP Fraud Service for the real-time fraud request and response.
2. GCP analytics consumer to the approved Azure messaging endpoint for asynchronous event consumption.

Network controls should restrict communication by the required source, destination, protocol, and port once the application interface is finalized.

On Azure, I would evaluate controls such as subnet segmentation, Network Security Groups, Azure Firewall where justified, and routing policy. On GCP, corresponding VPC firewall policies/rules and segmentation would control the GCP side.

The existence of a VPN route should never provide the GCP fraud or analytics environment with broad access to unrelated Azure databases, management interfaces, application subnets, or other banking workloads.

Likewise, an Azure workload should not receive broad access to unrelated GCP resources.

Logical separation between the synchronous fraud flow and asynchronous analytics flow should be maintained even if both use the same resilient cross-cloud connectivity infrastructure.

## Encryption and Key Management

The cross-cloud VPN provides network-layer encryption through IPsec/IKEv2.

For the fraud request itself, I would also require application/service-layer encryption, normally TLS. This protects the service interaction independently of the network tunnel.

The architecture therefore uses defense in depth:

**Service-Level TLS + Cross-Cloud IPsec Encryption**

The asynchronous messaging connection should likewise use encrypted service communication, and sensitive queued data should be protected at rest according to its classification.

Keys, certificates, and secrets should be managed through approved cloud key-management capabilities rather than embedded in source code or configuration files. Azure Key Vault and corresponding GCP key/secret-management capabilities are examples of services I would evaluate.

I would not automatically require customer-managed keys for every component. Key ownership and management requirements should follow data classification, regulation, organizational policy, and the threat model.

## Workload Identity and Authorization

Network location does not establish identity.

A request arriving through the VPN should not automatically be trusted because it originated from an approved network.

The real-time flow should require the Azure Payment Service to authenticate as a workload before the GCP fraud service authorizes it to submit fraud-evaluation requests.

Similarly, the GCP analytics consumer should authenticate as its own workload before receiving permission to consume approved Azure messages.

I would prefer workload identities and short-lived credentials or tokens over long-lived shared API keys or static secrets wherever the platforms and integration pattern support them.

Different functions should use different identities:

- Azure payment workload identity;
- GCP fraud-service identity;
- GCP analytics-consumer identity;
- human analyst identities; and
- privileged cloud/network administrator identities.

A fraud analyst should not automatically receive permission to change cross-cloud routing. A network administrator should not automatically receive access to payment data.

Privileged human access should use stronger controls such as MFA, role-based access, limited privileged roles, controlled elevation where appropriate, and audit logging.

The trust model becomes:

**Secure Network Path + Verified Identity + Explicit Authorization + Least Privilege**

## Timeout, Retry, Correlation, and Idempotency

Each real-time fraud request should carry a unique transaction/correlation identifier.

That allows the institution to trace a transaction across the cloud boundary:

`Azure request sent -> GCP request received -> fraud decision generated -> response returned -> Azure response received -> payment policy applied`

If Azure does not receive a response, it cannot assume that GCP never processed the request. The response may have been generated but lost during a connectivity failure.

Any retry behavior therefore needs to preserve the same transaction context and avoid treating a retry as an unrelated payment evaluation.

Retries should also be bounded by the remaining real-time processing window. If there is insufficient time for a controlled retry, the payment service should invoke the predefined timeout/fallback policy rather than consume the entire processing window with repeated requests.

The timeout itself does not mean the payment is fraudulent. It means the required external fraud decision was not available within the permitted window.

A representative audit event could record:

`Transaction ID | Fraud Request Timeout | 30 Seconds | Fallback Policy Applied`

without placing unnecessary sensitive customer data into logs.

## Durable Messaging and Backlog Recovery

For the asynchronous path, Azure-hosted durable messaging decouples event production from GCP availability.

If the GCP consumer or cross-cloud connectivity is unavailable, Azure continues retaining events according to the defined retention policy.

Operational monitoring should include:

- queue depth;
- age of the oldest unprocessed message;
- message processing rate;
- failed messages;
- dead-letter messages; and
- consumer availability.

After recovery, the GCP consumer should process the backlog in a controlled manner. Recovery testing should verify that events are not lost and that duplicate processing is handled appropriately.

This queue solves data preservation and recovery. It does not solve the real-time fraud-decision requirement for a payment that is already waiting for a decision.

## Monitoring and Observability

I would monitor the architecture at several layers because healthy infrastructure does not necessarily mean the business service is healthy.

### Connectivity Health

Monitor:

- VPN tunnel status;
- BGP session status;
- expected route exchange;
- failover state; and
- cross-cloud network availability.

### Security Health

Monitor for:

- unexpected denied cross-cloud connections;
- attempts to reach unauthorized networks;
- authentication failures;
- privileged configuration changes;
- routing or firewall changes; and
- unusual workload behavior.

### Application and Integration Health

Monitor:

- fraud requests sent;
- fraud requests received;
- response success/failure;
- fraud response latency;
- requests approaching the assumed 30-second threshold;
- requests exceeding the threshold; and
- correlation failures.

### Asynchronous Processing Health

Monitor:

- queue depth;
- oldest-message age;
- processing rate;
- consumer health; and
- dead-letter conditions.

A critical architecture principle is:

> **A healthy VPN does not prove that fraud decisions are being returned within the required payment-processing window.**

The technical telemetry should support a business-level service-health view, but executives should see service impact and continuity state rather than BGP sessions, firewall events, and raw cloud logs.

## Security Detection and Incident Response

The design should support containment rather than relying only on alerting.

For example, if a GCP analytics workload begins attempting connections to Azure networks it has never been authorized to reach, the firewall should block those attempts and the activity should be available for security investigation.

A response could progress through:

**Detect -> Correlate -> Assess -> Contain -> Recover -> Validate**

Depending on the incident, containment might include:

- revoking or disabling the affected workload identity;
- blocking the affected cross-cloud flow;
- isolating a compromised workload;
- preserving relevant logs/evidence;
- remediating or rebuilding the workload; and
- restoring access only after validation.

I would not automatically shut down the entire Azure-GCP VPN for every suspected compromise. Segmentation should allow the organization to isolate an affected workload or flow first, avoiding unnecessary disruption to the real-time fraud service when possible.

Security telemetry from both cloud environments should support centralized correlation through the organization's monitoring/SIEM capability. The architectural requirement is cross-cloud investigation and containment, not dependence on a particular SIEM vendor.

## Resilience and Failure Scenarios

Different failures require different responses.

### Single VPN Tunnel Failure

Redundant VPN paths and BGP should move traffic to a healthy path. Operations should be alerted, but the application should ideally experience no business interruption.

### Broader Cross-Cloud Connectivity Failure

For production, I would evaluate whether dedicated private connectivity should provide the primary path with VPN connectivity as an independent backup. The decision would depend on availability, latency, volume, recovery objectives, regulation, and cost.

### GCP Fraud Service Failure

If the network is healthy but the fraud service is unavailable, network failover does not solve the problem.

Azure waits only for the defined fraud-decision window. Under the case-study assumption, a decision not received within 30 seconds triggers the institution's predefined fraud-continuity policy.

The fallback policy might distinguish among transaction risk levels. The architecture should not assume that every timed-out payment is automatically approved or automatically fraudulent.

### GCP Asynchronous Consumer Failure

Events remain in the Azure durable queue and processing resumes after recovery. Queue health and backlog age provide operational visibility.

### Azure Payment Application Failure

Cross-cloud redundancy cannot compensate for failure of the Azure payment application itself. Production application availability and recovery need to be designed separately within Azure.

### Response Lost After GCP Processing

If GCP processes a request but Azure never receives the response, correlation and idempotency controls prevent the architecture from treating a controlled retry as a completely unrelated fraud request.

Retry behavior must still remain within the allowed processing window.

## Patch and Vulnerability Management

Secure architecture requires ongoing maintenance.

Cloud providers manage portions of the underlying platform under the shared-responsibility model. The financial institution remains responsible for customer-managed applications, configurations, dependencies, identities, operating systems/images where applicable, and other components within its responsibility.

I would use a vulnerability-management lifecycle such as:

**Discover -> Assess -> Prioritize -> Remediate -> Validate -> Document Exceptions**

Maintenance must also respect the availability design. Redundant components or paths should not be taken out of service simultaneously unless an approved maintenance plan explicitly accepts that risk.

A typical approach is:

**Verify Redundancy -> Shift/Remove One Component -> Patch/Update -> Validate -> Restore -> Repeat on Redundant Component**

Critical vulnerabilities may require remediation outside normal maintenance windows. Prioritization should consider exploitability, exposure, affected data, business impact, available compensating controls, and urgency.

Temporary exceptions should have a documented owner, justification, compensating controls, approval, and expiration/review date.

## Change Management, Configuration Drift, and Rollback

Architecture approval establishes a controlled baseline. It is not permanent approval for every future connectivity, access, or data-use change.

Material changes should answer:

- What is changing?
- Why is it changing?
- What is the security and business impact?
- How will it be tested?
- What demonstrates success?
- What conditions require the change to stop?
- How will the environment return to the known-good configuration?

For a routing change, for example, expected routes should be documented before and after implementation. If unintended networks become reachable, that should be treated as a failure criterion and the change should be rolled back.

Configuration drift should be distinguished from an approved change. Unexpected firewall, routing, IAM, or security-policy changes should be detectable against the approved baseline.

In a production environment, I would evaluate Infrastructure as Code and automated configuration-policy enforcement to improve repeatability, peer review, traceability, rollback, and drift detection.

This is a production architecture recommendation. The original Azure-GCP HA VPN PoC was primarily implemented using cloud CLI/portal-based configuration and should not be represented as having implemented production IaC governance.

Business and data changes also matter. If the fraud use case later requires additional sensitive customer information or expands into a new purpose, that should trigger data classification, privacy, security, and governance review rather than being treated as a simple interface update.

## Technical Validation and Evidence

I would validate the architecture against its requirements rather than assume controls work because they appear on a diagram.

### Connectivity Validation

- Intentionally take down a VPN tunnel.
- Verify BGP reconverges to a healthy path.
- Confirm expected routes remain available.
- Confirm unauthorized prefixes do not become reachable.

### Segmentation Validation

- Verify approved Azure-to-GCP and GCP-to-Azure flows succeed.
- Attempt an unauthorized path, such as GCP analytics to an Azure management network.
- Verify the connection is blocked and appropriately logged.

### Encryption Validation

- Verify the cross-cloud VPN uses the approved IPsec configuration.
- Verify service-to-service communication uses the required application-layer encryption.

### Identity Validation

- Verify valid workload identity succeeds.
- Verify invalid or expired identity fails.
- Verify one workload cannot assume another workload's permissions.
- Verify privileged administrative activity is logged.

### Real-Time Fraud Flow Validation

Trace a test transaction across the complete interaction using its correlation identifier:

`Azure Request -> GCP Received -> Decision Generated -> Response Returned -> Azure Received -> Payment Policy Applied`

Then make the fraud service unavailable and verify that the assumed 30-second timeout produces the expected fallback behavior rather than leaving the payment indefinitely waiting.

This is a production validation scenario for the hypothetical case study, not a test claimed to have been executed in the original PoC.

### Asynchronous Recovery Validation

- Stop the GCP consumer.
- Continue generating test events.
- Verify Azure retains the events.
- Observe queue depth and message age.
- Restore the consumer.
- Verify backlog processing completes without event loss or inappropriate duplication.

### Security Response Validation

Simulate an unauthorized cross-cloud access attempt and verify that it is blocked, logged, detected, and available to the appropriate security response process.

### Change and Rollback Validation

Perform a controlled configuration change, validate the expected outcome, and demonstrate restoration to the known-good configuration. In a mature production environment, also validate configuration-drift detection.

## Evidence I Would Expect

Depending on implementation and organizational policy, architecture validation evidence could include:

- architecture and data-flow diagrams;
- sanitized VPN and BGP status;
- sanitized route tables;
- firewall/NSG policy and test results;
- workload authentication evidence;
- correlation traces;
- queue and backlog metrics;
- alert evidence;
- failover test results;
- security-event evidence; and
- change/rollback records.

Evidence from the original PoC should remain clearly distinguished from validation that would be required for the hypothetical production banking architecture.

## Architecture Outcome

For this scenario, I would not treat secure multicloud connectivity as simply establishing a VPN between Azure and GCP.

The production architecture needs to protect a time-sensitive payment decision while limiting cross-cloud trust and containing failures.

The design therefore combines:

- redundant encrypted connectivity;
- constrained routing;
- deny-by-default segmentation;
- service-level encryption;
- workload identity and explicit authorization;
- data minimization and tokenization where appropriate;
- bounded fraud-decision timeout behavior;
- correlation and idempotency;
- durable asynchronous messaging;
- layered operational and security monitoring;
- cross-cloud incident containment;
- tested failover and recovery;
- vulnerability and patch management; and
- controlled change, rollback, and drift management.

The original Azure-GCP HA VPN project demonstrates the underlying resilient connectivity pattern. This case study shows the additional decisions I would need to evaluate before using that pattern to support a regulated, real-time payment workload in production.
