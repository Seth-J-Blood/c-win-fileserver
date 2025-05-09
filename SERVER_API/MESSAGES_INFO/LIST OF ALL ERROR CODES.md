All errors are negative 32-bit integers. One error code may be returned by multiple functions.

| ERROR NAME                       | ERROR VALUE  | ERROR MEANING                                                                                              |
| -------------------------------- | ------------ | ---------------------------------------------------------------------------------------------------------- |
| `ERROR_NO_FREE_SLOTS`            | `0xFFFFFFFF` | A user attempted to join the server, but there was no free space.                                          |
| `ERROR_NONBLOCKING_FAIL`         | `0xFFFFFFFE` | A socket could not be made non-blocking. The connection was shut down as a result.                         |
| `ERROR_INSUFFICIENT_PERMISSIONS` | `0xFFFFFFFD` | An operation failed because the permissions required by it were not high enough. See [[USER PERMISSIONS]]. |
| `ERROR_USER_INVALID`             | `0xFFFFFFFC` | An operation targeting an active client failed because the client is no longer active.                     |
| `ERROR_LOGIN_FAILED`             | `0xFFFFFFFB` | A user attempted to log into the server, but no username or password match was found.                      |
| `ERROR_BOOTED`                   | `0xFFFFFFFA` | The client has been booted from the server.                                                                |
| `ERROR_TIMEOUT`                  | `0xFFFFFFF9` | An operation or connection timed out, and has been stopped.                                                |
|                                  |              |                                                                                                            |
|                                  |              |                                                                                                            |
