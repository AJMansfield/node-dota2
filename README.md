node-dota2
========

A node-steam plugin for Dota 2, consider it in alpha state.

## Initializing
Parameters:
* `steamClient` - Pass a SteamClient instance to use to send & receive GC messages.
* `debug` - A boolean noting whether to print information about operations to console.

```js
var Steam = require('steam'),
    steamClient = new Steam.SteamClient(),
    dota2 = require('dota2'),
    Dota2 = new dota2.Dota2Client(steamClient, true);
```

## Methods
All methods require the SteamClient instance to be logged on.

### Steam
#### launch()

Reports to Steam that you're playing Dota 2, and then initiates communication with the Game Coordinator.

#### exit()

Tells Steam you were feeding.


### Inventory
#### setItemPositions(itemPositions)
* `itemPositions` - An array of tuples (itemid, position).

Attempts to move items within your inventory to the positions you set. Requires the GC to be ready (listen for the `ready` event before calling).

#### deleteItem(itemid)

Attempts to delete an item. Requires the GC to be ready (listen for the `ready` event before calling).


### Chat
#### joinChat(channel, [type])
* `channel` - A string for the channel name.
* `[type]` - The type of the channel being joined.  Defaults to `dota2.DOTAChatChannelType_t.DOTAChannelType_Custom`.

Joins a chat channel.  Listen for the `chatMessage` event for other people's chat messages.

#### leaveChat(channel)
* `channel` - A string for the channel name.

Leaves a chat channel.

#### sendMessage(channel, message)
* `channel` - A string for the channel name.
* `message` - The message you want to send.

Sends a message to the specified chat channel, won't send if you're not in the channel you try to send to.


### Guild
#### requestGuildData()

Sends a request to the GC for new guild data, which returns `openPartyData` events - telling the client the status of current open parties for each guild, as well as exposing `guildIds` to the client.


#### inviteToGuild(guildId, targetAccountId, [callback])
* `guildId` - ID of a guild.
* `targetAccountId` - Account ID (lower 32-bits of a 64-bit steam id) of user to invite to guild.
* `[callback]` - optional callback, returns args: `err, response`.

Attempts to invite a user to guild. Requires the GC to be ready (listen for the `ready` event before calling).

#### cancelInviteToGuild(guildId, targetAccountId, [callback])
* `guildId` - ID of a guild.
* `targetAccountId` - Account ID (lower 32-bits of a 64-bit steam id) of user whoms guild invite you wish to cancel.
* `[callback]` - optional callback, returns args: `err, response`.

Attempts to cancel a user's guild invitation; use this on your own account ID to reject guild invitations. Requires the GC to be ready (listen for the `ready` event before calling).

#### setGuildAccountRole(guildId, targetAccountId, targetRole, [callback])
* `guildId` - ID of a guild.
* `targetAccountId` - Account ID (lower 32-bits of a 64-bit steam id) of user whoms guild invite you wish to cancel.
* `targetRole` - Role in guild to have.
* * `0` - Kick member from guild.
* * `1` - Leader.
* * `2` - Officer.
* * `3` - Member.
* `[callback]` - optional callback, returns args: `err, response`.

Attempts to set a user's role within a guild; use this with your own account ID and the 'Member' role to accept guild invitations. Requires the GC to be ready (listen for the `ready` event before calling).


### Community
#### profileRequest(accountId, requestName, [callback])
* `accountId` - Account ID (lower 32-bits of a 64-bit Steam ID) of the user whose passport data you wish to view.
* `requestName` - Boolean, whether you want the GC to return the accounts current display name.
* `[callback]` - optional callback, returns args: `err, response`.

Sends a message to the Game Coordinator requesting `accountId`'s profile data. Provide a callback or listen for `profileData` event for Game Coordinator's response. Requires the GC to be ready (listen for the `ready` event before calling).

#### passportDataRequest(accountId, [callback])
* `accountId` - Account ID (lower 32-bits of a 64-bit Steam ID) of the user whose passport data you wish to view.
* `[callback]` - optional callback, returns args: `err, response`.

Sends a message to the Game Coordinator requesting `accountId`'s passport data. Provide a callback or listen for `passportData` event for Game Coordinator's response. Requires the GC to be ready (listen for the `ready` event before calling).

#### hallOfFameRequest([week], [callback])
* `[week]` - The week of which you wish to know the Hall of Fame members; will return latest week if omitted.  Weeks also randomly start at 2233 for some reason, valf please.
* `[callback]` - optional callback, returns args: `err, response`.

Sends a message to the Game Coordinator requesting the Hall of Fame data for `week`. Provide a callback or listen for the `hallOfFameData` event for the Game Coordinator's response. Requires the GC to be ready (listen for the `ready` event before calling).


### Matches
#### matchDetailsRequest(matchId, [callback])
* `matchId` - The matches ID
* `[callback]` - optional callback, returns args: `err, response`.

Sends a message to the Game Coordinator requesting `matchId`'s match details. Provide a callback or listen for `matchData` event for Game Coordinator's response. Requires the GC to be ready (listen for the `ready` event before calling).

Note:  There is a server-side rate-limit of 100 requests per 24 hours on this method.

#### matchmakingStatsRequest()

Sends a message to the Game Coordinator requesting some matchmaking stats. Listen for the `matchmakingStatsData` event for the Game Coordinator's response (cannot take a callback because of Steam's backend, or RJackson's incompetence; not sure which). Rqeuired the GC to be ready (listen for the `ready` event before calling).


### Lobbies
#### createPracticeLobby([gameName], [password], [serverRegion], [gameMode], [callback])
* `[gameName]` Display name for the lobby (optional).
* `[password]` Password to restrict access to the lobby (optional).
* `[serverRegion]` Server region for the lobby` see [ServerRegion Enum](#Enums) (optional).
* `[gameMode]` Gamemode for the lobby` see [GameMode Enum](#Enums)(optional).
* `[callback]` - optional callback` returns args: `err` response`.

Sends a message to the Game Coordinating requesting to create a lobby.  Provide a callback or listen for `practiceLobbyJoinResponse` for the Game Coordinator's response (Node:  GC seems to erroneously return `DOTA_JOIN_RESULT_ALREADY_IN_GAME`).  Requires the GC to be ready (listen for the `ready` event before calling).

#### leavePracticeLobby()

Sends a message to the Game Coordinator requesting to leave the current lobby.  Requires the GC to be ready (listen for the `ready` event before calling).



### Leagues
#### leaguesInMonthRequest([month], [year], [callback])
* `[month]` - Int for the month (MM) you want to query data for.  Defaults to current month. **IMPORTANT NOTE**:  Month is zero-aligned, not one-aligned; so Jan = 00, Feb = 01, etc.
* `[year]`  - Int for the year (YYYY) you want to query data for .  Defaults to current year.
* `[callback]` - optional callback` returns args: `err` response`.

Sends a message to the Game Coordinator requesting data on leagues being played in the given month.  Provide a callback or listen for `leaguesInMonthResponse` for the Game Coordinator's response.  Requires the GC to be ready (listen for the `ready` event before calling).


## Events
### `ready`
Emitted when the GC is ready to receive messages.

### `unready`
Emitted when the connection status to the GC changes, and renders the library unavailable to interact.


### `chatMessage` (`channel`, `senderName`, `message`, `chatObject`)
* `channel` - Channel name.
* `senderName` - Persona name of user who sent message.
* `message` - Wot u think?
* `chatObject` - The raw chat object to do with as you wish.

Emitted for chat messages received from Dota 2 chat channels

### `guildOpenPartyData` (`guildId`, `openParties`)
* `guildId` - ID of the guild.
* `openParties` - Array containing information about open guild parties.  Each object has the following properties:
* * `partyId` - Unique ID of the party.
* * `member_account_ids` - Array of account ids.
* * `time_created` - Something about Back to the Future.
* * `description` - A user-inputted string.  Do not trust.

Emitted for each guild the bot's account is a member of, containing information on open parties for each guild.  Also exposes guildId's, which is handy.

### `guildInvite` (`guildId`, `guildName`, `inviter`, `guildInviteDataObject`)
* `guildId` - ID of the guild.
* `guildName` - Name of the guild.
* `inviter` - Account ID of user whom invited you.
* `guildInviteDataObject` - The raw guildInviteData object to do with as you wish.

You can respond with `cancelInviteToGuild` or `setGuildAccountRole`.

### `profileData` (`accountId`, `profileData`)
* `accountId` - Account ID whom the data is associated with.
* `profileData` - The raw profile data object.

Emitted when GC responds to the `profileRequest` method.

See the [protobuf schema](https://github.com/SteamRE/SteamKit/blob/master/Resources/Protobufs/dota/dota_gcmessages.proto#2261) for `profileData`'s object structure.

### `passportData` (`accountId`, `passportData`)
* `accountId` - Account ID whom the passport belongs to.
* `passportData` - The raw passport data object.

Emitted when GC responds to the `passportDataRequest` method.

See the [protobuf schema](https://github.com/SteamRE/SteamKit/blob/master/Resources/Protobufs/dota/dota_gcmessages.proto#L2993) for `passportData`'s object structure.

### `matchData` (`matchId`, `matchData`)
* `matchId` - Match ID whom the data is associatd with.
* `matchData` - The raw match details data object.

Emitted when GC responds to the `matchDetailsRequest` method.

See the [protobuf schema](https://github.com/SteamRE/SteamKit/blob/master/Resources/Protobufs/dota/dota_gcmessages.proto#L2250) for `matchData`'s object structure.

### `hallOfFameData` (`week`, `featuredPlayers`, `featuredFarmer`, `hallOfFameResponse`)
* `week` - Week the data is associated with.
* `featuredPlayers` - Array of featured players for that week. `[{ accountId, heroId, averageScaledMetric, numGames }]`
* `featuredFarmer` - Featured farmer for that week. `{ accountId, heroId, goldPerMin, matchId }`
* `hallOfFameResponse` - Raw response object.

Emitted when the GC responds to the `hallOfFameRequest` method.

### `matchmakingStatsData` (`waitTimesByGroup`, `searchingPlayersByGroup`, `disabledGroups`, `matchmakingStatsResponse`)
* `waitTimesByGroup` - Current average matchmaking waiting times, in seconds, per group.
* `searchingPlayersByGroup` - Current players searching for matches per group.
* `disabledGroups` - I don't know how the data is formatted here, I've only observed it to be zero.
* `matchmakingStatsResponse` - Raw response object.

Emitted when te GC response to the `matchmakingStatsRequest` method.  The array order dictates which matchmaking groups the figure belongs to, the groups are discoverable through `regions.txt` in Dota 2's game files.  Here are the groups at the time of this sentence being written (with unecessary data trimmed out):

```
    "USWest":               {"matchgroup": "0"},
    "USEast":               {"matchgroup": "1"},
    "Europe":               {"matchgroup": "2"},
    "Singapore":            {"matchgroup": "3"},
    "Shanghai":             {"matchgroup": "4"},
    "Brazil":               {"matchgroup": "5"},
    "Korea":                {"matchgroup": "6"},
    "Austria":              {"matchgroup": "8"},
    "Stockholm":            {"matchgroup": "7"},
    "Australia":            {"matchgroup": "9"},
    "SouthAfrica":          {"matchgroup": "10"},
    "PerfectWorldTelecom":  {"matchgroup": "11"},
    "PerfectWorldUnicom":   {"matchgroup": "12"}
```

### `practiceLobbyJoinResponse`(`result` `practiceLobbyJoinResponse`)
* `result` - The result object from `practiceLobbyJoinResponse`.
* `practiceLobbyJoinResponse` - The raw response object.

Emitted when the GC responds to `createPracticeLobby` method; erroneously emits "DOTA_JOIN_RESULT_ALREADY_IN_GAME" though` so never trust it. vOv


### `leaguesInMonthResponse` (`result`, `leaguesInMonthResponse`)
* `result` - The result object from `leaguesInMonthResponse`.
* `leaguesInMonthResponse` - The raw response object.

Emitted when the GC responds to `leaguesInMonthRequest` method.

Notes:

* The `month` property is used to filter the data to the leagues which have matches scheduled in the given month, however the `schedule` object contains schedules for a league's entire duration - i.e. before or after `month`.
* `month` is also zero-aligned, so January = 0, Febuary = 1, March = 2, etc.
* Not every participating team seems to be hooked up to Dota 2's team system, so there will be a few `{ teamId: 0 }` objects for some schedule blocks.

The response object is visualized as follows:

```
{
    eresult,            // EResult enum
    month,              // Int representing which month this data represents.
    leagues: [{         // An array of CMsgLeague objects
        leagueId,       // ID of the league associated
        schedule: [{    // An array of CMsgLeagueScheduleBlock objects
            blockId,    // ID represending this block
            startTime,  // Unix timestamp of a scheduled match (or group of matches)
            finals,     // Boolean represending if this match is a final.
            comment,    // Comment about this scheduled block - often the team names & position in bracket
            teams: [{   // An array of CMsgLeagueScheduleBlockTeamInfo objects
                teamId, // ID of the associated team
                name,   // The teams name
                tag,    // The teams tag
                logo    // The teams logo
            }]
        }]
    }]
}
```

## Enums
### ServerRegion
* `UNSPECIFIED: 0`
* `USWEST: 1`
* `USEAST: 2`
* `EUROPE: 3`
* `KOREA: 4`
* `SINGAPORE: 5`
* `AUSTRALIA: 7`
* `STOCKHOLM: 8`
* `AUSTRIA: 9`
* `BRAZIL: 10`
* `SOUTHAFRICA: 11`
* `PERFECTWORLDTELECOM: 12`
* `PERFECTWORLDUNICOM: 13`

Use this to pass valid server region data to `createPracticeLobby`.

### GameMode
* `DOTA_GAMEMODE_NONE: 0` - None
* `DOTA_GAMEMODE_AP: 1` - All Pick
* `DOTA_GAMEMODE_CM: 2` - Captain's Mode
* `DOTA_GAMEMODE_RD: 3` - Random Draft
* `DOTA_GAMEMODE_SD: 4` - Single Draft
* `DOTA_GAMEMODE_AR: 5` - All Random
* `DOTA_GAMEMODE_INTRO: 6` - Unknown
* `DOTA_GAMEMODE_HW: 7` - Diretide
* `DOTA_GAMEMODE_REVERSE_CM: 8` - Reverse Captain's Mode
* `DOTA_GAMEMODE_XMAS: 9` - The Greeviling
* `DOTA_GAMEMODE_TUTORIAL: 10` - Tutorial
* `DOTA_GAMEMODE_MO: 11` - Mid Only
* `DOTA_GAMEMODE_LP: 12` - Least Played
* `DOTA_GAMEMODE_POOL1: 13` - Limited Heroes
* `DOTA_GAMEMODE_FH: 14` - Compendium
* `DOTA_GAMEMODE_CUSTOM: 15` - Unknown

Use this to pass valid game mode data to `createPracticeLobby`.

## Testing
There is no automated test suite for node-dota2 (I've no idea how I'd make one for the stuff this does :o), however there the `test` directory does contain a Steam bot with commented-out dota2 methods; you can use this bot to test the library.

### Setting up
* Copy `config_SAMPLE.js` to `config.js` and edit appropriately.
* Create a blank file named 'sentry' in the tests directory.
* Attempt to log-in, you'll receive Error 63 - which means you need to provide a Steam Guard code.
* Set the Steam Guard code in `config.js` and launch again.
