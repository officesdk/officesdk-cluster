# Office SDK Cluster · Self-hosted Shimo Document Platform

[简体中文](README.md) · English

Deploy the Shimo Document Platform in your own environment to add **Office file preview, collaborative editing, and format conversion** to applications such as intranets, contract management, and records systems. Users open documents inside the application they already use. Your application manages user identities and file permissions.

**[Product overview](docs/product.en.md) · [Integration](docs/integration.en.md) · [Deployment](docs/deployment.en.md) · [Operations](docs/operations.en.md)**

## Quick start

**Open a collaborative document from a contract details page.** An editor should be able to change the document, close it, and see the change on reopening. A reviewer with read-only access should be able to open the same document and be prevented from editing it.

- **If you need a platform instance**, start with a single-node deployment and verify the service endpoint. [Deployment guide](docs/deployment.en.md)
- **If you already have an instance**, configure application credentials and callbacks. Create the file from your backend and embed the editor in your frontend. [Collaborative editing guide](docs/integration.en.md)

## Choose an integration path

| What you want to do | Integration path |
| --- | --- |
| Show an uploaded Office attachment on an approval page | Keep the original in your storage and provide a preview based on the user's permissions. |
| Let multiple users revise a contract from its details page | Create a collaborative file and open the editor with the same `fileId` for each user. |
| Import a contract for editing and archive the final copy | Start import and export jobs, then pass the results to the next step once the jobs succeed. |

[See integration options and verification steps](docs/product.en.md)

## Versions and distribution

Shimo supplies the installer, release package, and license. The API examples follow the `co1.8` developer documentation. Check the version you receive for supported features, API fields, and deployment requirements.

[Shimo product site](https://shimo.net/products/office-sdk) · [Official developer documentation (Chinese)](https://support.shimo.net/apidoc/docs-site/6000018/doc-338241) · [Official deployment and operations documentation (Chinese)](https://support.shimo.net/apidoc/docs-site/6000017/338575m0)
