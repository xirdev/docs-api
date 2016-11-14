# **Token Service**

_token provides a token for signals authentication.

You may create a token specifically for a path or a universal token. The V2 API requires a full path (/domain/application/room) while the V3 API requires just requires a token.

To add a chatter key add k=chatter_key.

curl -s -XPUT "https://user:secret@endpoint/_token/my/path?k=chatter_key&expire=20" | jq .{"v": "ZG1enoNPMqSLf93auDFjMD84jTVRMsoor7nHHMGoG_4HsH3ogWlNUmMwIFekwe44FEWYoBQMBAFfiFluzOL2K6QZJ2Qc4H1-bCaYcYpuRB9Tvcqyv9l53RHG1SUHdL7tCEjxrDDFDMtDrRgs5A","s": "ok"}

This example, generates a API V2 compatible token which includes the path specified, and only allows login to the specified path. It is equivalent to the V2 API /getToken url.

curl -s -XPUT "https://user:secret@endpoint/_token?expire=20" | jq .{"v": "tAbfpN-kmrxtRIV6JBgjUVA1ZDeLtypCx-x1MuIHSmdsA2NRDXCwtZmgzrV5t_TBEByXUh-nXhZt8Mvof2YP9VkL9KJmg7X-2U2DzHZ31GJ9ZW3CxgJMcaYHrC0v689RA25fusceIsBJSGoN4ggupeqp","s": "ok"}

Generates a token without a path requirement.

An expiration of 0 is no expiration.