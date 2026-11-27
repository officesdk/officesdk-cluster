# Operations guide

[Home](../README.en.md) · [简体中文](operations.md) · English

## Find where the request fails

| Symptom | Check first | How to narrow it down |
| --- | --- | --- |
| Nobody can reach the platform | Check DNS, the certificate, and the service endpoint from the affected network. Then check whether service Pods are ready. | Compare the admin console, demo, and business application to distinguish a platform access issue from an integration issue. |
| The demo works, but the document fails to open from the business application | Check the file creation result, connectivity from the platform to both callbacks, the application configuration, and signature expiry. | Compare application and platform logs using the same timestamp and `fileId`. Verify which user the callback identifies. |
| The file opens in read-only mode | Compare access in the business application with the `readable` and `editable` values returned by the file callback. | Open the same file as an editor and as a read-only user, then compare the permissions each account receives. |
| Only preview of the original attachment fails | Check `type`, `ext`, and `downloadUrl` in the file callback. | Confirm the platform can download the original from its network. Compare with a known-good file of the same format. |
| The modified timestamp in the application list does not update after editing | Reopen the file and check its contents. Then see whether the application received, verified, and processed the update event. | Record the document save result separately from the event processing result. [Saving and update events](https://support.shimo.net/apidoc/docs-site/6000018/doc-338253) |

Confirm the cluster and namespace before checking Pod events and container logs. Substitute the values from your environment. If the container restarted, add `--previous` to `logs` to view the previous container's logs.

```sh
kubectl config current-context
kubectl -n "$NAMESPACE" get pods -o wide
kubectl -n "$NAMESPACE" describe pod "$POD"
kubectl -n "$NAMESPACE" logs "$POD" -c "$CONTAINER" --since=30m
```

Record the release version, incident time and time zone, affected scope, redacted file identifier, and first relevant error. Before sharing logs, remove user information, download credentials, signatures, tokens, and complete URLs containing authorization parameters. [Service logs](https://support.shimo.net/apidoc/docs-site/6000017/338373m0) · [Monitoring metrics reference](https://support.shimo.net/apidoc/docs-site/6000017/338875m0)

## Backup and recovery

- **Define what to back up.** Include MySQL, MongoDB, Redis, object storage, installation settings, and environment parameters in the same recovery plan. The application team backs up the file mappings, permission records, and originals it manages. Agree on acceptable data loss and recovery time before choosing a backup frequency, retention period, and off-site copy strategy.
- **Check the backups.** For bundled databases, check scheduled jobs against the manual for your release. For external databases, coordinate backups with their owners. Regularly verify the most recent successful backup, file size, readability, and replica location. Back up object storage separately and confirm that attachments can be recovered to a point consistent with the database.
- **Rehearse a restore.** Select a recovery point and restore databases, objects, and configuration in an isolated environment. Check an existing collaborative document, a file with an attachment, and the permissions of multiple users. Record how long recovery takes. Before restoring production, confirm the cluster, namespace, write freeze window, and which data will be overwritten. Take a fresh backup first. [Backup and restore procedures](https://support.shimo.net/apidoc/docs-site/6000017/338341m0)

## Upgrade and roll back

1. Record the current release, target release, and license scope. Use the matching upgrade package in a representative test environment. Back up databases, objects, and configuration before upgrading and record the recovery point.
2. Upload the package in the admin console's system upgrade page. Review changes to resources and configuration, paying particular attention to deletions and services that require a restart. Confirm the maintenance window, apply the upgrade, and keep the execution logs.
3. Once the services are ready, verify that you can open existing files, edit with two users, reopen a saved file, and enforce read-only access. Repeat any import or export workflows you have integrated. Record the results and the deployed version.
4. If the checks fail, keep the failure logs and assess a rollback using the pre-upgrade snapshot in the upgrade history. If you also need to restore business data, follow the rehearsed recovery procedure and account for writes since the upgrade. [System upgrades and version rollback](https://support.shimo.net/apidoc/docs-site/6000017/338375m0)
