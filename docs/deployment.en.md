# Self-hosted deployment and verification

[Home](../README.en.md) · [简体中文](deployment.md) · English

Use this guide to choose a deployment method, install the Shimo Document Platform, and hand over an environment ready for integration testing.

## Choose a deployment method

| Your environment | Where to start |
| --- | --- |
| First evaluation with one eligible server | Start with a single-node Kubernetes deployment with network access. [Single-node deployment](https://support.shimo.net/apidoc/docs-site/6000017/338578m0) |
| Multiple nodes required | Use the installer to create a new Kubernetes cluster. [Cluster deployment](https://support.shimo.net/apidoc/docs-site/6000017/338621m0) |
| Existing cluster, offline installation, or external middleware | Follow the deployment path for your release and check image, storage, and middleware requirements. [Official deployment documentation](https://support.shimo.net/apidoc/docs-site/6000017/338575m0) |

## Prepare for deployment

| Check | Single-node evaluation with network access | New cluster |
| --- | --- | --- |
| Nodes | One fresh server | At least three nodes. Verify network connectivity between them and synchronize their clocks. |
| Compute and storage | Reference sizing is 16 CPU cores, 32 GB RAM, 100 GB for the root partition, and 300 GB free on a separately mounted `/data` volume. | Use the single-node sizing for each node as a starting point. Plan capacity for your document volume and concurrency. |
| Network | The administrator can reach the installer. The business application can reach the platform domain. The platform can reach the application backend. | Also check that Kubernetes Pod and Service address ranges do not overlap with existing networks. |
| Installation files | An installer that matches the CPU architecture and the corresponding Kubernetes release package | Also collect node details, roles, and SSH connection information. |

The sizing above comes from the official environment reference. Check the operating system, CPU architecture, mount paths, ports, and resource requirements for your release. Configure the platform's domain name and HTTPS certificate, and restrict access to the installer console. [System requirements reference](https://support.shimo.net/apidoc/docs-site/6000017/338337m0)

## Run the installer

1. Obtain the installer and release package from Shimo. Upload the installer to the installation node. Start the web installer according to the instructions for your release and record the displayed URL.
2. Open the installer from your administration workstation. Select the release package saved on that workstation and check the verification result. Enter the platform domain, node and SSH details, namespace, data directory, and middleware options.
3. Review the target nodes and disks in the deployment summary, then run the environment checks. Fix failures and rerun the checks. Review what any remaining warnings mean for your environment before starting installation.
4. Record the admin console and demo URLs, platform domain, release version, and namespace. Store credentials in your organization's password manager. Follow the handover instructions to configure the license and application. [Deployment steps and installer instructions](https://support.shimo.net/apidoc/docs-site/6000017/338575m0)

## Verify the deployment and the integration

| Stage | What to do | What success looks like |
| --- | --- | --- |
| Deployment handover | Check nodes and workloads. Open the admin console and demo. | The nodes report `Ready`, service Pods are ready, and initialization jobs have completed successfully. An administrator can sign in and view machine and license information. |
| Application integration | Create, edit, and reopen the same file from the actual application using test accounts. Then try a read-only account. | The changes persist, collaborators see the same content, and the read-only account cannot edit it. If attachment preview or format conversion is in scope, verify those features with agreed sample files. |

Run these read-only checks from an administration environment connected to the target cluster. Replace `NAMESPACE` with the value recorded by the installer.

```sh
kubectl config current-context
kubectl get nodes
kubectl -n "$NAMESPACE" get pods,jobs
```

Record the environment, release version, verification time, document types, and any failed checks so you can repeat the same checks after upgrades. [Integrate your first document](integration.en.md) · [Operations guide](operations.en.md)
