# API integration guide

[Home](../README.en.md) · [简体中文](integration.md) · English

This guide uses a contract application to show how to create a blank collaborative document. An editor will be able to change the document from the contract details page, and a reviewer will be able to open the same file with read-only access. If you need to display an existing attachment, start with the preview section in this guide. [Preview an existing attachment](#preview-an-existing-attachment)

## Before you start

- Obtain a working platform endpoint. Your frontend and backend both need to reach the platform, and the platform needs to reach your application's callback endpoint. [Deployment and verification](deployment.en.md)
- In the MDP operations console, open **Document Platform → Application Management**. Add an authorized application, enable editing and the office suites you need, and enter the base URL for your backend callbacks. Save the configuration, record the `appId`, and store the `appSecret` on your backend. [Application management](https://support.shimo.net/apidoc/docs-site/6000017/338367m0) · [Core concepts](https://support.shimo.net/apidoc/docs-site/6000018/doc-338241)
- Prepare two test accounts with edit access, one read-only reviewer account, and one account without access. Use them to check collaboration and permissions.

## 1. Identify the file and the user

Assign a unique, persistent `fileId` to the test contract, such as `contract-demo-001`. Use it when creating the file, looking it up in callbacks, and opening the editor. Generate the `signature` with the application secret on your backend and issue a `token` for the signed-in user. Give the frontend only the short-lived credentials needed to open the file. Follow the endpoint documentation for the signature type, fields, and expiration time. Mask query strings containing credentials in logs. [Signature](https://support.shimo.net/apidoc/docs-site/6000018/doc-338246) · [Token](https://support.shimo.net/apidoc/docs-site/6000018/doc-338245)

## 2. Implement two callbacks on your backend

Each path is relative to your application's callback base URL. For these two callbacks, verify `X-Weboffice-Token`, identify the user, and look up their information and file permissions. [Callback overview](https://support.shimo.net/apidoc/docs-site/6000018/doc-338277)

| Request from the platform | What your backend returns |
| --- | --- |
| `GET /users/current/info` | The signed-in user's ID, name, and related fields. Switching to another user should return that user's information. [User information fields](https://support.shimo.net/apidoc/docs-site/6000018/api-3470944) |
| `GET /files/{fileId}` | File metadata including an `id` that matches the path, the file name, the type, and `permissions` for the current user. The type is `documentPro` in this example. Reviewers can read the file with read-only access, and users without access are denied. [Collaborative file metadata fields](https://support.shimo.net/apidoc/docs-site/6000018/api-3470935) |

## 3. Create the collaborative file from your backend

Replace the host with your platform domain and substitute the actual URL-encoded query values. Record the file ID and creation result in your test database, then reuse the ID when opening the file. [Official create-file example](https://support.shimo.net/apidoc/docs-site/6000018/doc-338711)

```http
POST /sdk/v2/api/files?appId=<appId>&signature=<signature>&token=<token> HTTP/1.1
Host: mdp.example.com
Content-Type: application/json

{"fileId":"contract-demo-001","type":"documentPro"}
```

## 4. Open the editor on the contract details page

Create a container with a defined height, such as `<div id="editor" style="height:80vh"></div>`. The `credentials` object contains the `fileId`, `signature`, and `token` returned by your backend for the current user. Add `weboffice-js-sdk` according to the documentation for your release, then use `connect` in your frontend. [Frontend API](https://support.shimo.net/apidoc/docs-site/6000018/doc-338257)

```js
import { connect } from 'weboffice-js-sdk';

export function openContract(credentials) {
  return connect({
    endpoint: 'https://mdp.example.com/sdk/v2',
    fileId: credentials.fileId,
    signature: credentials.signature,
    token: credentials.token,
    container: document.querySelector('#editor'),
  });
}
```

## 5. Verify saving and permissions

1. Open the contract as an editor, enter a sentence you can recognize, and wait for it to save. The second editor should be able to open the same `fileId`, see the change, and continue editing.
2. Close and reopen the document as both editors to check that the changes persist. The platform automatically saves collaborative content. If your application database needs to update the contract's last-modified time, implement `POST /events`, verify the event credential, and handle duplicate notifications idempotently. [Saving and event delivery](https://support.shimo.net/apidoc/docs-site/6000018/doc-338253)
3. Open the document as the read-only reviewer and confirm that editing is disabled. Open it as the unauthorized user and confirm that access is denied. If the accounts receive the same permissions, check which user the callback identifies and the `permissions` it returns. [Permission design](https://support.shimo.net/apidoc/docs-site/6000018/doc-338252)

## Preview an existing attachment

Keep the original in your application storage. In the file callback, return `type: "file"`, the `ext` without a leading dot, `permissions.readable` for the current user, and a `downloadUrl` the platform can reach. Use `connect` with `mode: 'preview'` in your frontend. The official example uses a new `fileId` when the source file changes and needs a fresh preview. [Preview integration example](https://support.shimo.net/apidoc/docs-site/6000018/doc-338710) · [Preview file metadata](https://support.shimo.net/apidoc/docs-site/6000018/api-3470936)

For additional methods, parameters, and responses, see the backend API reference. If the deployment checks pass and the application page still fails to open, use the operations guide to isolate the failing connection. [Backend API](https://support.shimo.net/apidoc/docs-site/6000018/doc-338267) · [Operations guide](operations.en.md)
