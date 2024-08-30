1. We have deployed a POD hosting an application. Inspect it. Wait for it to start.

```shell
k get po
NAME       READY   STATUS    RESTARTS   AGE
webapp-1   1/1     Running   0          12s
```

2. A user - USER5 - has expressed concerns accessing the application. Identify the cause of the issue.

Inspect the logs of the POD

Account Locked due to Many Failed Attempts.

```shell
k logs webapp-1 | grep USER5
[2024-08-30 12:51:43,339] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
```

3. We have deployed a new POD - webapp-2 - hosting an application. Inspect it. Wait for it to start.

```shell
k get po
NAME       READY   STATUS    RESTARTS   AGE
webapp-1   1/1     Running   0          3m33s
webapp-2   2/2     Running   0          11s
```

4. A user is reporting issues while trying to purchase an item. Identify the user and the cause of the issue.

Inspect the logs of the webapp in the POD

USER 30 - Item Out of Stock

```shell
k logs webapp-2 | less
[2024-08-30 12:55:35,678] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
```