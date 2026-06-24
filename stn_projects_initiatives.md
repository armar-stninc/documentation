# STN Projects, Tasks & Initiatives

## GPU One

### Projects / Tasks / Initiatives

1. **Bare Metal Provisioning** — We currently use MAAS to provision GPU One servers. It is extensible and can support just about any process or workflow we need, and it exposes an API that we target for automated workflows. MAAS handles the following:
   - BMC IP address assignment
   - Server IP address assignment
   - **Commissioning** — load an ephemeral image to catalog and store hardware profiles, run hardware health checks, update BIOS settings, configure storage, update NIC firmware settings, and run custom scripts before the OS is installed
   - **Install base operating system** — we currently deploy Ubuntu 22.04 and 24.04, but custom images (via Packer) are supported
   - **Machine lifecycle** —
   
   Trevor is currently working on a project to replace this process.

2. **GPUONE Runner** — This is the process we currently use to provision GPU One servers. The script runs every five seconds and checks NetBox to determine whether a server's provisioning status has been set to "provisioning." If set, it starts the provisioning process on the matching servers. This script can definitely be streamlined. The script can also be configured to limit the concurrency. The full workflow can be found here: https://lucid.app/lucidchart/90a1e01d-fc1b-42bd-86e9-d3639454e197/edit?invitationId=inv_c3f05aca-8e41-4de3-bf4d-d3f5307097cc&page=l5DolkhWV5MN#

3. **Update MAAS to use the internal Linux repo.** - Trevor is/has a working repo now

4. **Healthcheck Script** — We currently use a script in a tenant environment that checks for total GPUs, GPU errors, missing configuration, network connectivity, and other metrics specific to the tenant and hardware type. We need to consolidate these into a single script that can be used across any tenant environment, with the understanding that each tenant environment will have unique aspects.

5. **Kubernetes** — We currently offer a managed Kubernetes service based on RKE2 (Rancher). Deployment of Kubernetes on nodes is done through a script (repo: TBD). We deploy the latest version, but will deploy whichever version the customer wants above v1.32. We handle deployment of the network operator, GPU operator, and MetalLB. We need to turn this into a true product (operationalize it), including automated deployment/provisioning and customer documentation. Documentation should make clear where our responsibility starts and stops.

6. **Slurm** — We provide a managed Slurm product. This also needs to be operationalized with customer documentation. It is co-managed to a degree, so we need to document the roles and responsibilities between STN and the customer. We will use the latest version of Slurm (25.11 as of now) unless the customer requires an older version. I've created playbooks to streamline the deployment of the Slurm master, SlurmDBD, MySQL, and login node. These playbooks need to be updated for use across more than one customer — i.e., leveraging `group_vars` — before they're ready for production use. We also need to integrate the new custom munge `.deb` package we created to address the critical security issue.

7. **Slurm on Kubernetes** — I've been working on "Slurm on Kubernetes" using the Slinky project (https://github.com/slinkyproject) as a base. This lets us easily deploy Slurm and automatically add each Kubernetes GPU node into the Slurm cluster, as well as create associated partitions; login nodes are also created in the cluster and allow SSH access. This is considered beta and needs more work before we can use it for customers. Our GitHub repo: TBD.

8. **Netris** — This product facilitates multi-tenant networking, including configuring both the switches and the host/server networking — specifically backend networking. We define the endpoints, and Netris algorithms automatically generate and apply the correct configurations across all network elements. Netris has a VM appliance called SoftGate that provides elastic IPs, NAT, tenant-specific connectivity, and L4 load balancing — functionality that switches alone cannot provide. Netris is a costly expense that I believe we can duplicate in house.

9. **Oxide** — We will begin using Oxide as our compute platform going forward to host customer and infrastructure workloads. Rack-mount servers will continue to be used as supplemental compute as needed.

10. **Always-Ready POC Environment** — We need to solidify the POC environment so that it's always ready to use. It should take no more than an hour to prep one or more GPU nodes (wiping, provisioning, network configuration, storage provisioning).

11. **Storage** — Provisioning storage for customer workloads needs to be fully automated. Automation is needed for NetApp, Pure Storage FlashArray, Pure Storage FlashBlade, and Pure Storage FlashBlade EXA.

12. **Documentation** — As these services are operationalized (Kubernetes, Slurm, compute), we need to ensure good customer documentation exists.

13. **SLA Reporting and Monitoring** — Transmute Data has done a lot of work aligning SLA requirements in the contracts with SLA-affecting metrics from Prometheus. The SLA defined in the contract is written for lawyers. The expectation we've gathered from customers with SLAs we must meet is that a node is either usable or not:
    - For Kubernetes customers, if the node is cordoned, it's not usable and counts as downtime.
    - For non-Kubernetes / Slurm / bare-metal customers, if the node is in drain, it's not usable and also counts as downtime.

    This also includes frontend and backend networking. The healthcheck script tracks these metrics and pushes them into Prometheus via the node exporter. Scheduled maintenance does not affect downtime and is excluded from the calculation, but is shown to the customer for visibility. I've also created a script in `/opt/monitor` on the customer's management server that runs on a schedule and tracks available/not-available status with the reason, time period, and total downtime. We use this data to determine credits back to the customer.

14. **Acceptance Testing** — There's an acceptance testing document in Dropbox → STN Team Folder → Document Library → SOP that we should use after cluster deployment, and whose artifacts can be turned over to the customer.

15. **Trusted / Confidential Computing** — This topic comes up in conversations with potential clients. It provides an end-to-end solution for bootstrapping hardware-rooted cryptographic trust for remote machines, the provisioning of encrypted payloads, and run-time system integrity monitoring. It also provides a flexible framework for the remote attestation of any given PCR (Platform Configuration Register). Users can create their own customized actions that trigger when a machine fails its attested measurements. We've looked at Keylime as a solution to address this but have not completed testing.

16. **RMA** — Develop a comprehensive RMA process. This will include diagnosis, vendor communication, artifact gathering and delivery (logs, any vendor-specific details), ticketing, follow-up, and tracking. Transceiver replacements should all be tracked and documented; we need to understand what our MTTR and MTBF are.

17. **Automated Node Replacement** - GPU clusters will have spare nodes as part of the buildout. We need to automate the process of replacing a bad node with a spare node. This would include lifecycle reporting, auto-generated tickets, RMA workflow processes, node replacement, burn-in, validation and acceptance.

---

## MSP

### Linux Repositories

We need to create hosted Linux repos that the provisioning system and customer environments can target for consistent installs and updates. Trevor set up a repo for hosting packages/software, but we need a true mirror. A repo update strategy needs to be developed and documented, which will be especially important for GPU One customers.

1. Ubuntu 22.04
2. Ubuntu 24.04
3. Rocky Linux 9

### FedRAMP / PCI / Security Compliance

We need to ensure that systems in scope adhere to FedRAMP and PCI controls.

1. **vCenter** — move to the management VLAN.

2. **Jump box(es)** — used for accessing management devices. Access via RBAC/SSO, including MFA/ZeroTrust.

3. **Network segmentation** — limit access to the management network to the jump box. Rules should be in place to limit traffic to/from devices that need access.

4. **DMZ configuration** — The CH2 DMZ has been configured, with a load balancer in place and rules to limit access. Change-control processes need to be developed and followed for traffic 
policies. A DMZ zone exists in SV7 but is not being used; we need to move the SV7 load balancers (with associated firewall policies) to the DMZ.

5. **Compliance strategy** — Complete the overarching PCI/FedRAMP compliance strategy.

6. **MDM** — Implement an MDM solution.

### Internal

1. **Patching and vulnerability remediation** — review and formalize the process.

2. **Standardize OS / OS upgrades** — Review operating systems installed across the environment to confirm none are deprecated or EOL. I've seen CentOS 7 still in the environment that needs to be upgraded.

3. **Ninja RMM** — The Ninja client needs to be installed across the environment. This is critical to address patching, inventory, and remediation.

4. **SentinelOne** — SentinelOne needs to be installed on every endpoint across the environment. We are currently out of compliance. Kartavya has, and can provide, a report on systems out of compliance to be remediated. If the Ninja RMM client is installed, SentinelOne should get installed automatically — we need to verify this is the case. We will be transitioning to XSIAM in the near future, so we need to prepare for moving assets from SentinelOne to XSIAM.

5. **Internal laptop provisioning** — Need a formalized process for Mac and Windows laptops: asset tags, LOB applications, security software (EDR and RMM). MFA should be enabled for all users on login. Full-disk encryption must be enabled on all endpoints.

6. **Capacity planning** — Revisit and fix the capacity planning strategy/policy. We are currently reactive to capacity issues instead of proactive. We need to monitor Cohesity, hypervisor datastores, hypervisor hosts, Pure Storage (FlashArray, FlashBlade, FlashBlade EXA), NetApp, and Wasabi. This strategy should cover both storage and compute resources. We should always have enough compute and tier-1/primary storage to support our largest DRaaS customer, as well as maintain an N+2 HA cluster configuration.

7. **Monitoring and observability** — We use Grafana/Prometheus for GPU One. Zabbix is used internally and for customers (via Zabbix proxies) for monitoring and alerting. We've discussed migrating away from Zabbix to Grafana/Prometheus and need to get forward movement on this.

8. **Intune** - Leverafe intune for Windows Autopilot, streamlining new device provisioning, including applications. Can we leverage MDM and MAM across windows/mac?

### VDI as a Service

We use Parallels to provide VDI to customers. This also needs to be operationalized (process, documentation, etc.). GoldenBear is currently the only customer using our VDI-as-a-Service product. Today it's nothing more than RDP over VPN, and due to some customer policies the experience has not been great — their VDI-hosted VMs become unavailable and require a reboot to fix. We need to set up Parallels controllers, add VDI machines to them, publish machines to tenant users, and configure the applicable firewall policies to turn this into a proper production-ready service. Customer documentation also needs to be created for the client installation workflow.

### Backup as a Service

We use Cohesity as our platform for BaaS, backing up both internal and customer resources. Cohesity is designed for multi-tenant workflows, supporting a wide range of products and protecting VMs (VMware, Hyper-V, Nutanix), Oracle, MSSQL, file, and NAS. The product can also host file services (which we use for customers) and archive backup data to S3-compatible object storage; our Archive-as-a-Service is based on Wasabi. Cohesity is security-focused and encrypts data at rest and in transit. Each storage domain (a storage partition on the Cohesity cluster) is created per customer, along with a unique customer encryption key.

For **every** cluster deployed, monitoring must be configured and alerts sent to a monitored email such as support@stninc.com or baas-support@stninc.com and/or slack. Tickets should be created from these emails so the systems team can remediate.

### DR as a Service

We use Cohesity as our platform for DRaaS. Customers subscribed to this service use STN as the target for disaster recovery. We fill out a Disaster Recovery worksheet that outlines how the DR process works — i.e., priority for restoring VMs, resources required, and RPO/RTO objectives. We've developed a set of scripts (which require RVTools from the customer environment so that MAC addresses can be restored) that programmatically restore the virtual machines from backup, add the associated network adapters and adapter types, and power on the VMs. This runs off the Cohesity storage. During a DR test, this is fine and is the documented way to complete a DR test. Cohesity storage is not very performant, but in a DR situation it provides the customer with close to immediate access to their running VMs. In a true disaster, the VMs would be migrated to primary storage for performance.

This process needs improvement, as the scripts were written just to get the machines restored. Resilience needs to be addressed, and the process should be fully automated. The VMs should be restored and then added to the portal so the customer can access the console and power controls.

### STN ONE Portal

1. **Backup-as-a-Service module** — Customers need to see backup jobs, completion state (pass/fail), SLA objectives, backup policies, and restore-from-backup. We did some work with Enformion to show restored VMs, but need to expand that work to all customers and include the other features. It would be nice to make this module somewhat generic in case we decide to replace Cohesity as our backup platform or add additional data protection solutions.

2. **Oxide** — We're bringing in Oxide as the compute platform for GPU One deployments. We'll need to integrate the Oxide API with the Portal API that customers will target to manage resources.

3. **Terraform Provider** - Some work by Nick has been done on this. Need to complete the work.

4. **Kubernetes** - Need to revisit this module. Was not working 100%

4. **Virtual Machines** - Needs to be revamped. Deploying a virtual machine has a 80% success rate.
