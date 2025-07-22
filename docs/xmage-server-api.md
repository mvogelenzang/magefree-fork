# XMage Server API

This document describes the purpose of the server component used by XMage and details the methods exposed to clients via the `MageServer` interface.

The server is responsible for user management, match creation, tournament control and game play coordination. Clients communicate with it using remote calls where most methods require an authenticated `sessionId` provided after successful login. Some operations (e.g. finding a chat by table id) do not currently use a session id as indicated in the interface.

A session is established by calling `connectUser` (or `connectAdmin`) which returns `true` when authentication succeeds. The returned `sessionId` must be used in subsequent calls. Users may also restore a previous session using the `restoreSessionId` parameter. Administrators connect with a special password using `connectAdmin`.

## API Overview

Below is a short explanation for every method of `MageServer`. Parameters are listed in the order they appear in the interface and the return column shows the expected return type. Exceptions thrown are omitted for brevity.

| Method | Parameters | Returns | Behavior |
| --- | --- | --- | --- |
| `authRegister` | sessionId, userName, password, email | boolean | Register a new user account. |
| `authSendTokenToEmail` | sessionId, email | boolean | Send password reset token to the given email. |
| `authResetPassword` | sessionId, email, authToken, password | boolean | Reset a user's password using a previously emailed token. |
| `connectUser` | userName, password, sessionId, restoreSessionId, version, userIdStr | boolean | Authenticate and establish a player session. |
| `connectAdmin` | password, sessionId, version | boolean | Authenticate administrator access. |
| `connectSetUserData` | userName, sessionId, userData, clientVersion, userIdStr | boolean | Attach client metadata after login. |
| `ping` | sessionId, pingInfo | boolean | Keep the session alive and check connectivity. |
| `serverAddFeedbackMessage` | sessionId, username, title, type, message, email | void | Submit a feedback message to the server. |
| `serverGetPromotionMessages` | sessionId | Object | Retrieve promotion banner data. |
| `getServerState` | – | ServerState | Query current state information (number of users, etc.). |
| `serverGetMainRoomId` | – | UUID | Get the id of the default room. |
| `roomGetUsers` | roomId | List<RoomUsersView> | List users connected to the room. |
| `roomGetFinishedMatches` | roomId | List<MatchView> | Retrieve finished matches for the room. |
| `roomCreateTable` | sessionId, roomId, matchOptions | TableView | Create a new match table. |
| `roomCreateTournament` | sessionId, roomId, tournamentOptions | TableView | Create a new tournament table. |
| `roomJoinTable` | sessionId, roomId, tableId, name, playerType, skill, deckList, password | boolean | Join an existing match table. |
| `roomJoinTournament` | sessionId, roomId, tableId, name, playerType, skill, deckList, password | boolean | Join an existing tournament table. |
| `deckSubmit` | sessionId, tableId, deckList | boolean | Submit a deck list for the current table. |
| `deckSave` | sessionId, tableId, deckList | void | Save a deck to the server. |
| `roomWatchTable` | sessionId, roomId, tableId | boolean | Begin watching a match in progress. |
| `roomWatchTournament` | sessionId, tableId | boolean | Begin watching a tournament. |
| `roomLeaveTableOrTournament` | sessionId, roomId, tableId | boolean | Leave the specified table or tournament. |
| `tableSwapSeats` | sessionId, roomId, tableId, seatNum1, seatNum2 | void | Swap two seats at the table. |
| `tableRemove` | sessionId, roomId, tableId | void | Delete the given table. |
| `tableIsOwner` | sessionId, roomId, tableId | boolean | Determine if the calling user owns the table. |
| `roomGetTableById` | roomId, tableId | TableView | Get information about a single table. |
| `roomGetAllTables` | roomId | List<TableView> | List all tables within the room. |
| `chatSendMessage` | chatId, userName, message | void | Post a chat message. |
| `chatJoin` | chatId, sessionId, userName | void | Join a chat channel. |
| `chatLeave` | chatId, sessionId | void | Leave a chat channel. |
| `chatFindByGame` | gameId | UUID | Find chat id associated with a game. |
| `chatFindByTable` | tableId | UUID | Find chat id associated with a table. |
| `chatFindByTournament` | tournamentId | UUID | Find chat id associated with a tournament. |
| `chatFindByRoom` | roomId | UUID | Find chat id associated with a room. |
| `matchStart` | sessionId, roomId, tableId | boolean | Start a created match. |
| `matchQuit` | gameId, sessionId | void | Concede the given match. |
| `gameJoin` | gameId, sessionId | void | Join a running game. |
| `gameGetView` | gameId, sessionId, playerId | GameView | Retrieve the current game view for a player. |
| `gameWatchStart` | gameId, sessionId | boolean | Start watching a game. |
| `gameWatchStop` | gameId, sessionId | void | Stop watching a game. |
| `sendPlayerUUID` | gameId, sessionId, data | void | Send a UUID value from the client to the server. |
| `sendPlayerString` | gameId, sessionId, data | void | Send a string value from the client to the server. |
| `sendPlayerBoolean` | gameId, sessionId, data | void | Send a boolean value from the client to the server. |
| `sendPlayerInteger` | gameId, sessionId, data | void | Send an integer value from the client to the server. |
| `sendPlayerManaType` | gameId, playerId, sessionId, data | void | Send a chosen mana type. |
| `sendPlayerAction` | playerAction, gameId, sessionId, data | void | Perform an in-game action such as priority or concede. |
| `sendDraftCardPick` | draftId, sessionId, cardId, hiddenCards | DraftPickView | Pick a card during drafting. |
| `sendDraftCardMark` | draftId, sessionId, cardId | void | Mark a card during drafting. |
| `tournamentStart` | sessionId, roomId, tableId | boolean | Start a tournament. |
| `tournamentJoin` | draftId, sessionId | void | Join a draft or tournament. |
| `tournamentQuit` | tournamentId, sessionId | void | Concede from a tournament. |
| `tournamentFindById` | tournamentId | TournamentView | Look up a tournament by id. |
| `draftJoin` | draftId, sessionId | void | Join a draft queue. |
| `draftQuit` | draftId, sessionId | void | Leave a draft. |
| `draftSetBoosterLoaded` | draftId, sessionId | void | Confirm booster load for draft. |
| `replayInit` | gameId, sessionId | void | Prepare to replay a finished game. |
| `replayStart` | gameId, sessionId | void | Start game replay mode. |
| `replayStop` | gameId, sessionId | void | Stop game replay mode. |
| `replayNext` | gameId, sessionId | void | Advance one step in replay. |
| `replayPrevious` | gameId, sessionId | void | Go back one step in replay. |
| `replaySkipForward` | gameId, sessionId, moves | void | Skip a number of moves forward in replay. |
| `cheatShow` | gameId, sessionId, playerId | void | Reveal hand or other cheat functionality. |
| `adminGetUsers` | sessionId | List<UserView> | Retrieve list of connected users. |
| `adminDisconnectUser` | sessionId, userSessionId | void | Disconnect a specific user session. |
| `adminEndUserSession` | sessionId, userSessionId | void | Terminate a user session. |
| `adminMuteUser` | sessionId, userName, durationMinutes | void | Temporarily mute a user. |
| `adminLockUser` | sessionId, userName, durationMinutes | void | Temporarily lock a user account. |
| `adminActivateUser` | sessionId, userName, active | void | Set whether a user is active. |
| `adminToggleActivateUser` | sessionId, userName | void | Toggle a user's active state. |
| `adminTableRemove` | sessionId, tableId | void | Remove a table administratively. |
| `adminSendBroadcastMessage` | sessionId, message | void | Broadcast a message to all users. |
