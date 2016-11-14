# The Accounts Service

The accounts service provides 2 functions:

1. New logins to the base Xirsys account, that may be subsequently used in API interactions, i.e. other admin credentials. All facilities on your account are available to the new admin, and each change to the data is stamped with the admins credentials. You can see changes to data by using the time_series=1 option on most GET calls.

2. You may create full Xirsys sub-accounts, that are manipulated via the standard API.

## Administrators

### Create an Admin

Post a login name and password to /users/admins, for an administrator login.

`curl -H "Content-type: application/json" "http://user:secret@endpoint/_acc/users/admins" -d '{"k": "key", "v": {"password": "password"}}'`

### Delete an Admin

`curl -X DELETE "http://user:secret@endpoint/_acc/users/admins?k=key"`

### List Admins

`curl "http://user:secret@endpoint/_acc/users/admins"`

### Get Admin

`curl "http://user:secret@endpoint/_acc/users/admins?k=key"`

### Time Series

`curl "http://user:secret@endpoint/_acc/users/admins?k=key&time_series=1"`

### Usage

To login as an administrator, just use the following syntax in the login credentials.

`curl "https://user:admin_key#admin_password@endpoint/_data/my/path"
`
Notice that the secret is now loaded with 2 fields of information separated by a "#".

We need to have the main Xirsys account holder username (ident), and once we have that we can authenticate the admin within that context.

Currently all admins have the same rights, role based permissions will be available at some stage.

Note: Time series are returned with the time, login and value, so you can see who made changes to the data.

## Account Proxy

The _acc service allows the creation of full Xirsys sub-accounts that you may manage using the standard API.

This gives you fully self contained stats and services for your sub-accounts. For example, a sub-account could require it's own proprietary namespace.

### Create

Simply PUT to the _acc services /accounts path to add a new sub-account.

`curl -H "Content-type: application/json" "http://user:secret@endpoint/_acc/accounts" -d '{"k": "username", "v": {"password": "password","email": "email","first_name": "first_name","last_name": "last_name"}}'`

### Delete

This will delete the sub-account and all it's data.

Note: Sub-account data is completely self contained in it's own 'table', the entire 'table' is removed with this action.

`curl -XDELETE "http://user:secret@endpoint/_acc/accounts?k=username'
`
### Get

To get the details of a sub-account ...

`curl "http://user:secret@endpoint/_acc/accounts?k=username'
`
### Usage

Usage of sub-accounts is driven entirely through the overloading of the authorization headers. This allows the API to remain consistent, and to allow Xirsys V2 services to use the feature without extension.

The authorization of a sub-account must take place in the context of the primary Xirsys account holder (you), thus the primary username must always be present. The authorization header looks like this

primary_username#subaccount_username:subaccount_secret

Or more easily remembered, you must prefix primary_username# to any sub-account call.

For example to add a namespace to the sub-account

`curl -s -XPUT "https://primary_username#subacount_username:subaccount_secret@endpoint/_ns/my/path"`

Effectively this syntax redirects the call to a different database 'table', and this is consistent across all services.

For primary accounts executing requests for sub-accounts looking up a sub-account secret is not efficient, so primary account holders may authenticate with their own secret.

To indicate primary account holder secret prefix the secret with "!".

`curl -s -XPUT "https://primary_username#subacount_username:!primaryaccount_secret@endpoint/_ns/my/path"`

## Walkthrough

Here's a walk thru of using accounts, which includes creation of a top level account,via the sys user; I've included that so that you can see how your account creation is exactly the same as the system's

Here's the creation of a top level primary account by sys…

`curl -s -XPUT -H 'Content-type: application/json' "http://%24sys:secret@endpoint/_acc/accounts" -d '{"k": "elena", "v": {"email": "elena@user.cl","password": "woot"}}'{"v":{"username":"elena","term":"monthly","secret":"8af2a710-eba7-11e5-b4a0-9cdd20b4c6b5","plan":"free","password":"woot","last_name":null,"first_name":null,"email":"elena@async.cl","customer_id":null,"created":1458154052,"company":null,"active":true},"s":"ok"}
`
Notes:

* the key is the "ident", added to the body as "username" automatically

* email is required, and is added as a secondary index for the path "/accounts"

* active is true, controls if the user may login in or not

So, now, Elena is a top level account holder, and sys can query for her in 2 ways by ident(key), and by email, like this …

`curl -s  "http://%24sys:secret@endpoint/_acc/accounts?k=elena" | jq . {   "v": {     "username": "elena",     "last_name": null,     "first_name": null,     "email": "elena@async.cl",     "created": 1458203041,     "company": null,     "active": true   },   "s": "ok" }`

or because a secondary key was added automatically on email …

`curl -s  "http://%24sys:secret@endpoint/_acc/accounts?k2=elena@async.cl" | jq .{   "v": {     "username": "elena",     "last_name": null,     "first_name": null,     "email": "elena@async.cl",     "created": 1458203041,     "company": null,     "active": true   },   "s": "ok"}
`
Notice we're querying on k2.

### Aside: Secondary Keys

KV4 has 4 keys in total you can use to index into your objects per_path;k, k2, k3 and k4. So, accounts objects in /accounts could be looked up in 4 different ways, but the same is true for any other path you could create. These 4 keys are also integrated into the caching system so GETs on any of these are fast. 4 keys is not a hard limit, so more work could go into this to get more out of it. 4 keys was chosen as KV4 should have 4 indexes, we can add a fifth for kv5 :).

Please see Secondary Keys for more info on creation (TBD).

### Primary Account Holder Creates Secondary Account

So, account holder Elena wishes to create a secondary account. She can do so by PUTing to her /accounts, like this …

`curl -s -XPUT -H 'Content-type: application/json' "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_acc/accounts" -d '{"k": "lily", "v": {"email": "lily@async.cl","password": "woot2"}}' | jq .{  "v": {    "username": "lily",    "term": "monthly",    "secret": "068564e6-ec1e-11e5-b3d6-09c98c1c18d8",    "plan": "free",    "password": "woot2",    "last_name": null,    "first_name": null,    "email": "lily@async.cl",    "customer_id": null,    "created": 1458204940,    "company": null,    "active": true  },  "s": "ok"}`

So Elena PUTs sub account lily to her /accounts path in the _acc service. Notice the complete symmetry between the sys user putting a new account and Elena.

### Querying Sub Accounts

Just as sys would do to get a list of it's accounts, Elena may query her /accounts path,

`curl -s  "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_acc/accounts" | jq . { "v": [   "lily"  ],  "s": "ok"}`

so now we know lily is a sub account holder we can query for more details …

`curl -s  "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_acc/accounts?k=lily" | jq .{  "v": {    "username": "lily",    "last_name": null,    "first_name": null,    "email": "lily@async.cl",    "created": 1458204940,    "company": null,    "active": true  },  "s": "ok"}`

### Setting Sub Account Properties

You may alter sub account properties by POSTing new attribute values to the key. For example, let's say you want to change the "active" property ...

`curl -s -XPOST -H 'Content-type: application/json' "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_acc/accounts" -d '{"k": "lily", "v": {"active": false}}' | jq .{  "s": "ok"}
`
and to check …

`curl -s  "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_acc/accounts?k=lily" | jq .{  "v": {    "username": "lily",    "last_name": null,    "first_name": null,    "email": "lily@async.cl",    "created": 1458204940,    "company": null,    "active": false  },  "s": "ok"}
`
You may post new properties to the object using this means too. The system (should not) does not allow you to change email or username at this time.

### Lily as a Stand Alone User

Now that Lily has been created by Elena, she may want to add new namespaces for herself, so the new Lily account is completely self contained. So this may seem reasonable …

`curl -s -XPUT "http://lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@endpoint/_ns/a/path/that/lily/created" | jq .{  "v": "unauthorized",  "s": "error"}`

however, this would look Lily up in the top level accounts database (sys database), where it does not exist. The Xirsys top level database, only knows about Elena, not her secondary accounts. Thus, we need to perform the action in the context of Elena, so this seems reasonable …

`curl -s -XPUT "http://elena#lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@endpoint/_ns/a/path/that/lily/created" | jq .{  "v": "unauthorized",  "s": "error"}`

This would typically work, however, above, I deactivated the account, and the activation is checked in all authorizations, so, let's fix that …

`curl -s -XPOST -H 'Content-type: application/json' "http://elena:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_acc/accounts" -d '{"k": "lily", "v": {"active": true}}' | jq .{  "s": "ok"}
`
now ...

`curl -s -XPUT "http://elena#lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@endpoint/_ns/a/path/that/lily/created" | jq .{"s": "ok"}`

and to check to the 5th level …

`curl -s "http://elena#lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@endpoint/_ns/?depth=5" | jq .{  "v": [    "a/path/that/lily/created"  ],  "s": "ok"}`

### Usage - A Proxy Service.

The secondary account system purpose is to allow proxy usage of Xirsys. Elena can update her own accounts and data, but now she has a secondary account holder Lily, who she wants to create a Xirsys service for.

Thus a means for Elena to create namespaces, subscriptions and so on in the context of Lily is required.

Elena doesn't want to check Lily's secret each time, so she wants to do the authorization as herself with her secret, this may seem reasonable …

`curl -s -XPUT "http://elena#lily:9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_ns/a/path/that/elena/created/for/lily" | jq .{"v": "unauthorized","s": "error"}`

Here we have Elena trying to add a path for Lily but with her own secret, and it fails. Obviously, as above, the system assumes, that the secret provided is the user's (Lily's) secret and not Elenas. So to indicate that it's the primary account holders secret and not the secondary, we prefix secret with a "!", like so …

`curl -s -XPUT 'http://elena#lily:!9a3b4066-ec19-11e5-930a-2aee0ed40b9c@endpoint/_ns/a/path/that/elena/created/for/lily' | jq .{  "s": "ok"}`

Please note here, I changed the " to ' quotes for bash as it will do a substitution on the ! otherwise.

The data although authorized on Elena's account, does in fact go into Lily's data bucket. Here is Lily authorizing with Lily's secret, to find the new data deposited by the primary account holder …

`curl -s "http://elena#lily:068564e6-ec1e-11e5-b3d6-09c98c1c18d8@endpoint/_ns/?depth=7" | jq .

{  "v": [    "a/path/that/elena/created/for/lily","a/path/that/lily/created"  ],  "s": "ok"}
`

**All other services will authorize in the same way.**
