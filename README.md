UselessWords v1
===============

Concepts
--------
There are 2 concepts we care about sharing: accounts and posts. We also care about server keys, because security.

### Accounts
An account should be made available under the endpoint `<instance>/@<username>`. They must be accesible through json, HTML support is suggested, ideally displaying that user's public posts.

The account should look as follows:

```json
{
  "uid": "<username>@<instanceUrl>",
  "username": "upto20-urlsafe-chars",
  "displayName": "Some arbitrary text no longer than 80 characters",
  "description": "More arbitrary text. Limited to something sane like IDK 200 characters or something.",
  "avatar": "https://example.com/someimage.png",
  "createdAt": 1493262499,
  "updatedAt": 1493262499
}
```

A unique ID can be constructed: `<username>@<instance>`

Accounts can change only their display name, description, and avatar. Differences in username/uid should be treated as a new account.

### Posts

Posts will live beneath a user `https://<instance>/@<username>/<id>`. They must be accesible through json, HTML support is suggested.

A post should look as follows:
```json
{
  "id": "doesntReallyMatterMuch",
  "author": "user@example.com",
  "content": "This is a #stupid idea, right @terribleplan?",
  "userReferences": ["terribleplan@nrd.li"],
  "visibility": "public",
  "createdAt": 1493262499
}
```

A unique id can be constructed: `<id>@<username>@<instance>`.

A post can be a reply, in which case the optional `replyingTo` field should be set to a unique ID.

A post can be a repost, in which case the optional `reposting` field should be set to a unique ID.

Visibility is an enum of public/followers/instance/direct. The semantics of each are probably what you expect.

Posts are immutable (unless deleted).

### Keys

A server should provide a list of keys it may use to sign outgoing messages at `/keys`. It only really makes sense to use JSON here.

```json
{
  "<key-id>": "Public key contents, base64 encoded"
}
```

Keys should be cached, but not forever. Maybe a day?

Communication/Federation
------------------------

### Simple

The simplest way instances can talk to each other is by posting json to each other (basically webhooks).

If foo@bar.com follows terribleplan@nrd.li then `bar.com` notifies `nrd.li` that it wants updates from terribleplan.

```
POST https://nrd.li/subscribe

{
  "usernames": ["terribleplan"],
  "url": "https://bar.com/webhook/users"
}
```

nrd.li will then post updates requests to the provided URL. Those updates will be signed using a JSON web signature. THe following is an example of what the decoded request would look like.

```
POST https://bar.com/webhook.users

{
  "alg": "ES256",
  "kid": "<instanceUrl>:<ID>"
}.{
  "accounts": {
    "terribleplan": {
      "posts": [{
        "id": "chooseSomeSchemeAndUseItThisIsJustAStringUnder255Chars",
        "author": "user@example.com",
        "content": "This is a #stupid idea, right @terribleplan?",
        "userReferences": ["terribleplan@nrd.li"],
        "visibility": "public",
        "createdAt": 1493262499
      }]
    }
  }
}.thisWouldBeASignatureAccordingToJWS
```

bar.com would verify the update against the keys `nrd.li` makes available, then react once it verifies the message.

Notes
-----

* Only SSL, because otherwise you can't trust other servers, not really.
* Still need thought put into privacy/following. And maybe a way to avoid the admin of an instance being able to read those supposedly private posts. Crypto would have a bunch of overhead and need to be client-side. Eww.
* Probably want a way to revoke hook signing keys immediately in case of compromise.
* I think this is better since it's a push system instead of a push then pull system ala ostatus.
* Can use unique hook URLs a (weak) layer of defense.
* How to push account updates (`accounts.<user>.self` with the json)?
* How to do media?
* Maybe want a way to mark people as admins, mods, etc. (think instance-level verification)
* Most other things are implementation details.
* A more efficient way to communicate would be nice. HTTP has the benefit of allowing load balancing. Tradeoffs exist.
* I hope people hate this idea.
