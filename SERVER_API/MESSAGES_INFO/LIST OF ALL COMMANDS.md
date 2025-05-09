A "command" in this context refers to a message that may be sent to the server or to the client that informs the device that an action should be performed (or at least attempted). There are two types of commands: **backend commands** and **frontend commands**. A valid command is at most 75 bytes long, including the command header and `END_CMD` byte. Any command larger than 75 bytes is assumed to be garbage. 

>[!note] NOTE: "Ignored" commands still count against a user's `MAX_INSTRUCTIONS_PER_CYCLE`.
### BACKEND COMMANDS
**Backend commands** are commands that only a device should be able to produce. A backend command should not be the direct result of a user typing in a command and pressing enter. Some examples of backend commands are:
* `VERIFY_SELF - [0x01600D] | [END_CMD]`
	**ONLY SENT FROM CLIENT TO SERVER**: When sent by an unverified user, server now recognizes the connected client as a valid client running the correct client code. If an already-verified client sends this command again, it is ignored.
	
 * `LOGIN - [0x1066] | [encrypted_username (25 BYTES)] | [encrytped_password (40 BYTES)] | [reserved (7 BYTES)] [END_CMD]`
	**ONLY SENT FROM CLIENT TO SERVER**: When sent by a verified (but not enabled) client, the server decrypts the sent fields using its 256-byte AES key, compares it to the (encrypted) file contents of the corresponding username (if there is one). If there is no such username match or the password does not match, the server returns `ERROR_LOGIN_FAIL`.

* `DROP_CONNECTION - [0xDC] | [error_code (4 BYTES)] | [END_CMD]`
	**CAN BE SENT BOTH WAYS**: When sent to a device, the receiving device knows the connection to the sending device has been terminated (usually as a result of an unrecoverable error). The receiving device shuts down the connection on their end, not waiting for the send buffer to empty.

* `HEALTH_CHECK - [0xCC] | [END_CMD]`
	**CAN BE SENT BOTH WAYS**: When sent to a connected client, the connected client sends a `HEALTH_CHECK` command back. If no response is received within 15 seconds, the client is sent a `DROP_CONNECTION` command with error code `ERROR_TIMEOUT`, and the connection to the client is closed. The server will automatically a `HEALTH_CHECK` command every 10 seconds to every connected client. If a client goes 30 seconds without processing a `HEALTH_CHECK` or any other command from the server, it sends a `DROP_CONNECTION` command with error code `ERROR_TIMEOUT` to the server.


### FRONTENT COMMANDS