These are the various user permissions, including temporary and non-customizable permissions. This list also lays out the default permissions of each user level.
For every connected client, there are multiple fields defining how the client interacts with the server and what they can use:
* `MAX_IDLE_CYCLES`: the maximum amount of wait cycles where the user has not performed any command that can occur before they are automatically booted from the server to save resources.
* `MAX_UPLOAD_BYTES`: the maximum amount of bytes that can be uploaded (as a file) to the server per minute.
* `MAX_INSTRUCTIONS_PER_CYCLE`: the maximum amount of commands that the server can execute from the user in a single wait cycle (approx. 5ms).
* `PERMISSION_LEVEL`: the permission level of the user. The higher this value is, the more actions the user can perform (such as promoting or booting users). The maximum value of this field is 255.

## SERVER (NON-ASSIGNABLE)
>[!note] This role cannot be assigned to a connected client
>This permission level represents the permission level of console-executed commands from the server. 

The server. Nothing has a higher permission level than it, meaning any permission-dependent commands such as `BOOT_USER` or `PROMOTE_USER` automatically succeed. This role cannot be assigned to a connected client, and is not really a role, as it is just used to represent the server in documentation. Has a `PERMISSION_LEVEL` of 255.

 **Promoting users to the administrator role must occur through the server console.**

## ADMIN (SERVER ASSIGNABLE)
An administrator is a client authorized to do everything except demote/boot/ban other administrators. The server is the only thing that can demote an admin once they have been promoted. They have access to every command by default. This role is extremely dangerous and should only be given to trusted users who know what they are doing. By default a user with this role:
* Has a `MAX_IDLE_CYCLES` of `inf`.
* 

Has a `PERMISSION_LEVEL` of 200.
## MODERATOR (SERVER/ADMIN ASSIGNABLE)



## POWER USER (SERVER/ADMIN/MODERATOR ASSIGNABLE)



## ENABLED USER (ASSIGNED BY DEFAULT AFTER LOGIN)



## VERIFIED USER (ASSIGNED AFTER VERIFICATION)



## UNVERIFIED USER (ASSIGNED UPON CONNECTION)