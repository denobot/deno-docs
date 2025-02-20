# Key Expiration

:::caution Deno KV is currently in beta

Deno KV is currently **experimental** and **subject to change**. While we do our
best to ensure data durability, data loss is possible, especially around Deno
updates.

Excuting Deno programs that use KV currently requires the `--unstable` flag, as
below:

```sh
deno run -A --unstable my_kv_code.ts
```

:::

Since version 1.36.2, Deno KV supports key expiration. This allows an expiration
timestamp to be associated with a key, after which the key will be automatically
deleted from the database:

```ts,ignore
const kv = await Deno.openKv();

// `expireIn` is the number of milliseconds after which the key will expire.
function addSession(session: Session, expireIn: number) {
  await kv.set(["sessions", session.id], session, { expireIn });
}
```

Key expiration is supported on both Deno CLI and Deno Deploy.

## Atomic expiration of multiple keys

If multiple keys are set in the same atomic operation and have the same
`expireIn` value, the expiration of those keys will be atomic. For example:

```ts,ignore
const kv = await Deno.openKv();

function addUnverifiedUser(
  user: User,
  verificationToken: string,
  expireIn: number,
) {
  await kv.atomic()
    .set(["users", user.id], user, { expireIn })
    .set(["verificationTokens", verificationToken], user.id, { expireIn })
    .commit();
}
```

## Caveats

The expire timestamp specifies the _earliest_ time after which the key can be
deleted from the database. An implementation is allowed to expire a key at any
time after the specified timestamp, but not before. If you need to strictly
enforce an expiration time (e.g. for security purposes), please also add it as a
field of your value and do a check after retrieving the value from the database.
