# 接口接入指南

[返回首页](../README.md) · 简体中文 · [English](integration.en.md)

本指南以合同系统为例，从创建一份空白协同文档开始。完成后，经办人可以在合同详情页修改文档，审批人可以用只读账号打开同一份文件。已有附件需要预览时，可直接查看本文的附件预览章节。

## 联调前准备

- 取得可访问的中台服务地址。业务前端和后端都要能访问中台，中台也要能访问业务回调地址。[私有部署与验收](deployment.md)
- 在 MDP 运维平台进入「文档中台 → 应用管理」，新增授权应用，勾选编辑能力和所需套件，填写业务后端的回调根地址。保存后记录 `appId`，在业务后端保管 `appSecret`。[应用管理](https://support.shimo.net/apidoc/docs-site/6000017/338367m0) · [基本概念](https://support.shimo.net/apidoc/docs-site/6000018/doc-338241)
- 准备两个有编辑权限的测试账号、一个只读审批账号和一个无访问权的账号，用于验证多人协作和权限。

## 1. 为文件和用户确定身份

业务后端为测试合同分配一个唯一且持久的 `fileId`，如 `contract-demo-001`。创建、回调查询和打开编辑器都使用这个 ID。后端用应用密钥生成 `signature`，并为本次登录用户签发 `token`。前端只拿到打开文档所需的短期凭证。签名类型、字段和有效期按所调用的接口选取，日志中屏蔽带凭证的查询串。[Signature](https://support.shimo.net/apidoc/docs-site/6000018/doc-338246) · [Token](https://support.shimo.net/apidoc/docs-site/6000018/doc-338245)

## 2. 在业务后端实现两个回调

下表的路径都相对于**业务应用的回调根地址**。对于这两个用户操作回调，业务后端校验 `X-Weboffice-Token`，再以识别出的用户查询资料和文件权限。[回调总览](https://support.shimo.net/apidoc/docs-site/6000018/doc-338277)

| 中台发出的请求 | 业务后端应返回什么 |
| --- | --- |
| `GET /users/current/info` | 当前登录人的用户标识、姓名等资料。换一个用户打开时，返回值也应切换。[用户信息字段](https://support.shimo.net/apidoc/docs-site/6000018/api-3470944) |
| `GET /files/{fileId}` | 与路径一致的 `id`、名称、类型和当前用户的 `permissions` 等元信息。此例类型为 `documentPro`。只读审批人可读且不可编辑，无访问权的用户应被拒绝。[协同文件元信息字段](https://support.shimo.net/apidoc/docs-site/6000018/api-3470935) |

## 3. 由业务后端创建协同文件

以下是 HTTP 请求示意。主机换成中台域名，查询参数替换成 URL 编码后的实际值。在测试业务库中保留这次使用的文件 ID 和创建结果，后续打开时复用。[官方创建示例](https://support.shimo.net/apidoc/docs-site/6000018/doc-338711)

```http
POST /sdk/v2/api/files?appId=<appId>&signature=<signature>&token=<token> HTTP/1.1
Host: mdp.example.com
Content-Type: application/json

{"fileId":"contract-demo-001","type":"documentPro"}
```

## 4. 在合同详情页打开编辑器

页面准备一个有高度的容器 `<div id="editor" style="height:80vh"></div>`。下面的 `credentials` 代表业务后端为当前用户返回的 `fileId`、`signature` 和 `token`。在已按交付文档引入 `weboffice-js-sdk` 的前端项目中调用 `connect`。[前端 API](https://support.shimo.net/apidoc/docs-site/6000018/doc-338257)

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

## 5. 验证保存和权限

1. 经办人打开合同，输入一句可识别的文字，等待保存完成。第二名有编辑权的用户打开相同 `fileId`，应能看到并继续编辑。
2. 两人关闭页面再打开，核对文字仍在。协同内容由中台自动保存。若业务库还要更新合同修改时间，按需实现 `POST /events`，校验事件凭证并对重复通知做幂等处理。[保存与事件机制](https://support.shimo.net/apidoc/docs-site/6000018/doc-338253)
3. 换只读用户打开，核对内容可读且无法编辑。换无权用户打开，核对访问被拒绝。若两人权限相同，先查回调识别的用户和返回的 `permissions`。[权限设计](https://support.shimo.net/apidoc/docs-site/6000018/doc-338252)

## 如果需要的是已有附件预览

原件留在业务存储中。文件回调返回 `type: "file"`、不带点的 `ext`、当前用户的 `permissions.readable` 和中台能访问的 `downloadUrl`。前端可用同一个 `connect` 并指定 `mode: 'preview'`。按官方预览示例，原件内容变更且需要重新生成预览时使用新的 `fileId`。这条路径可以直接从一份现有附件开始。[预览接入示例](https://support.shimo.net/apidoc/docs-site/6000018/doc-338710) · [预览文件元信息](https://support.shimo.net/apidoc/docs-site/6000018/api-3470936)

更多方法、参数和响应见[中台后端 API](https://support.shimo.net/apidoc/docs-site/6000018/doc-338267)。部署通但页面打不开时，按[运维手册](operations.md)的调用方向排查。
