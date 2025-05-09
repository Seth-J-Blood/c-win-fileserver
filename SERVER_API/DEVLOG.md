
# USER SLOT SYSTEM
To keep track of all connected clients, S.E.T.H. utilizes a slotted system, where an array of "slots" (called `USER_SLOTS`) gets filled up and emptied by clients joining and leaving. If a client joins, the first available slot is filled, making it unusable by new clients until that client leaves, which frees up the slot. The server has a preset amount of slots it can hold, and if all slots are filled, no new client can join (they are immediately dropped and sent the 32-bit error code [[LIST OF ALL ERROR CODES|ERROR_NO_FREE_SLOTS]]). The server keeps track of which slots are filled by using a bitfield (called `FREE_USER_SLOTS`), with one bit for every client slot. If a client joins, the bit corresponding to the free slot is set. If a client leaves, the bit corresponding to their slot is cleared. 

There is also an array of 1-byte numbers called `USER_SESSION_IDS`, which has a byte for every user slot. When a client joins, a new thread is assigned to them to handle inputs and outputs. This thread has a copy of the 1-byte `SESSION_ID` that was set when the thread was created, and will terminate if the copy of the `SESSION_ID` does not equal the value stored in `USER_SESSION_IDS`. The `SESSION_ID` for a slot is incremented every time a user leaves, which terminates the listener thread. The `USER_SESSION_IDS`array was introduced to combat [[DEVLOG#DUPLICATE USER-THREAD FROM RACE CONDITION|a race condition]].

# INSTRUCTION PARSER
The thread attached to every connected client reads the client input into an 100 byte buffer. The stream of data is then parsed for commands (valid commands are assumed to be within 75 bytes long, including command header and `CMD_END`). To ensure no commands are dropped, the parser has two fields, `parseHead` and `parseTail`. `parseHead` contains the index of the `END_CMD` of the last successfully executed command, and `parseTail` contains the index of the last read byte. With these two fields, the number of bytes read is easily attainable, and the parser knows which bytes can be safely discarded when reading more incoming messages or disposing of garbage. To successfully parse instructions, the parser has four states:
* **PARSING**
	Active when the parser is actively reading new bytes from the stream. If the total read bytes exceeds 75 without finding `END_CMD`, the parser moves to state **DISPOSING**. Otherwise, if a command's header and `END_CMD` are found, the parser will move to state **EXECUTING**. If 75 bytes could not be read before finding `END_CMD` (and there is not any more data to be read), the parser will enter the **WAITING** state. If any junk values (nonvalid parameters, random numbers between command headers) are found, they are ignored.
* **EXECUTING**
	Active only when the parser has found a valid command, with a good header, `END_CMD`, and command length of at most 75 bytes. Parameters are not checked for validity during parsing, so the executor must handle malformed commands and invalid parameters. A [[DEVLOG#TO-DO COMMAND WEIGHT SYSTEM|command weighting system]] may be implemented to prevent the spam of heavy commands.
* **WAITING**
	Active when the parser is waiting for more data (ex. to complete a half-sent command), or while the parser's per-cycle [[DEVLOG#TO-DO COMMAND WEIGHT SYSTEM|weight limit]] has been exceeded. While in this state, the parser does nothing and just waits until the next cycle, where it re-checks if any data has been sent.
* **DISPOSING**
	Active only when the parser has received a junk command where the distance to the command header and the `END_CMD` exceeds 75. If this occurs, any values in the socket are ignored until another valid command header is read. If this state ever occurs, the user has their `SUSPICION_FLAG` set to true, and if this flag was already true, the server is told to boot the user.

# FILE METADATA INFORMATION
There is a file called "\_filedata.mtdata" for every file-holding folder that contains metadata for each file within. For every file, the following will be stored:

| FIELD SIZE/TYPE    | FIELD NAME         | DESCRIPTION                                                                                                               |
| ------------------ | ------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| `100 BYTES`        | `FILENAME`         | The name of the file that this metadata represents. Is null-terminated unless the filename string fills the whole buffer. |
| `UINT32 (4 BYTES)` | `VERSION`          | The version of the file. Every time the file is edited, this counter goes up by one.                                      |
| `UINT8 (1 BYTE)`   | `PERMISSION_LEVEL` | The permission level required to view, open, or edit the file.                                                            |
| `256 BYTES`        | `SHA256_HASH`      | The 256-byte SHA hash code for the file. Used to detect corruption in the file's data.                                    |
| `151 BYTES`        | `RESERVED`         | Reserved for future use. Should always be zero.                                                                           |

# DEVELOPER NOTES

# FIXED/IMPLEMENTED ITEMS
Items that have been successfully implemented, that were either on the [[DEVLOG#TO-DOs|TO-DO list]] or the [[DEVLOG#BUGS/SECURITY FLAWS|Bugs list]].

### DUPLICATE USER-THREAD FROM RACE CONDITION
>[!success] FIXED
>To fix, the server creates a new array `USER_SESSION_IDS` that contains a `UINT8` for every user slot. When a client leaves, the corresponding `UINT8` is incremented by one. The thread will detect this change and terminate. Using such a large value (255 different sessions before overflow) prevents any possible race conditions occurring where enough clients join and leave to reset the session ID to its previous value.

There is a small chance for a bad race condition to occur when a client leaves/is booted. To handle all incoming client traffic, the server utilizes POSIX-style threads, paired with a small delay time (usually 5ms) to reduce resource consumption. If a client leaves, the thread attached to them needs to terminate - but if a new client connects and takes up the same slot in the `USER_SLOTS` array as the old booted client within the old thread's delay time, the old thread will not detect that the client has been booted and will not terminate.

# BUGS/SECURITY FLAWS
Flaws that

### SERVER OVERLOADING SECURITY FLAW
>[!bug] UNFIXED
> To limit this, the server only parse and perform about 5 commands (frontend or backend) per cycle. This limit can be increased for users of higher permission (and trust) levels. If the client is simply sending garbage (meaning the command limit is not incremented), the `SUSPICION_FLAG` will be set for that user. If the `SUSPICION_FLAG` is already true and more garbage is sent, the user will be booted. "Garbage" in this case refers to junk or unexpected values, and the `SUSPICION_FLAG` will be set if over 50 bytes of garbage is sent. If 5 commands are successfully parsed without any garbage values (at all), the `SUSPICION_FLAG` can be set to false again.

There is a security flaw where a user could spam the server with tons of garbage and bog down resources. The server would attempt to parse all the garbage, burning resources and (if an expensive operation is spammed), potentially stalling the entire server. 
* Idea: [[DEVLOG#TO-DO COMMAND WEIGHT SYSTEM|Command weighting system]]


# TO-REMEMBERs
Things to keep in mind when writing code for S.E.T.H.

### TO REMEMBER: RW_WRLOCK SANITY CHECKS
>[!warning] When making changes to a shared resource, make sanity checks before changing anything to prevent any changes occurring between a `read_unlock` and a `write_lock`.
 For example, if a piece of code reads that something has correct permissions to make a change, and then performs a `read_unlock` and then a `write_lock` on the `RW_LOCK`, something may have changed in the time it unlocked and re-locked the `RW_LOCK`, so it needs to double check upon writing to ensure this didn't happen.


# TO-DOs
Things that have not been implemented nor are essential for security, but still need to get done before full release.

### TO-DO: DOWNLOAD/UPLOAD FILE RECOVERY
>[!question] To be implemented

If a user is uploading or downloading a large file and loses connection mid-download, a progress recovery system could be made available:
* To start: the following fields are added to the end of a file that has been interrupted:

| FIELD TYPE/SIZE | FIELD NAME        | DESCRIPTION                                                                                                     |
| --------------- | ----------------- | --------------------------------------------------------------------------------------------------------------- |
| `"STARTREC"`    | `RECOVERY_HEADER` | Marks the beginning of the recovery data area.                                                                  |
| `255 BYTES`     | `FILEPATH`        | The file path to the file that was being downloaded.                                                            |
| `8 BYTES`       | `UNIX_TIME`       | The number of seconds elapsed since the `UNIX_EPOCH` when the file download/upload was interrupted              |
| `256 BYTES`     | `SHA256_HASH`     | The 256-byte SHA hash code for the file at the time it was being downloaded, used to check for version changes. |
| `"ENDREC"`      | `RECOVERY_FOOTER` | Marks the end of the recovery data area.                                                                        |
If the in-progress file (file extension ".inprog") disagrees with the file stored on the server, the file download will have to be restarted.

### TO-DO: HANDLE CONCURRENT FILE CHANGING/DOWNLOADING/VIEWING
>[!question] To be implemented?


### TO-DO: COMMAND WEIGHT SYSTEM
>[!question] To be implemented?

Not all commands are created equal; some are very expensive for the server to complete, while others are done near-instantaneously. In addition to the [[DEVLOG#SERVER OVERLOADING SECURITY FLAW|maximum commands per cycle limit]], a command may have a "weight" applied to it, where executing it in a cycle counts as multiple commands rather than just one. For example, a heavy command may have a weight of 4, meaning that at most, that command and one small, 1-weight command can be run that cycle, while a light command may only have a weight of 1 or 2, meaning multiple other commands can be run in addition to the light one in one cycle. If an instruction is parsed and there is not enough "room" for the command to be executed (ex. a heavy command that has 4 weight is run, and then another command with weight 2 is parsed, exceeding the weight limit of 5), the thread will just put it off until the next cycle.