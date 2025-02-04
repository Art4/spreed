# Chat API

Base endpoint is: `/ocs/v2.php/apps/spreed/api/v1`

## Receive chat messages of a conversation

* Method: `GET`
* Endpoint: `/chat/{token}`
* Data:

    field | type | Description
    ------|------|------------
    `lookIntoFuture` | int | `1` Poll and wait for new message or `0` get history of a conversation
    `limit` | int | Number of chat messages to receive (100 by default, 200 at most)
    `timeout` | int | `$lookIntoFuture = 1` only, Number of seconds to wait for new messages (30 by default, 60 at most)
    `lastKnownMessageId` | int | Serves as an offset for the query. The lastKnownMessageId for the next page is available in the `X-Chat-Last-Given` header.

* Response:
    - Status code:
        + `200 OK`
        + `304 Not Modified` When there were no older/newer messages
        + `404 Not Found` When the conversation could not be found for the participant

    - Header:

        field | type | Description
        ------|------|------------
        `X-Chat-Last-Given` | int | Offset (lastKnownMessageId) for the next page.

    - Data:
        Array of messages, each message has at least:

        field | type | Description
        ------|------|------------
        `id` | int | ID of the comment
        `token` | string | Conversation token
        `actorType` | string | `guests` or `users`
        `actorId` | string | User id of the message author
        `actorDisplayName` | string | Display name of the message author
        `timestamp` | int | Timestamp in seconds and UTC time zone
        `systemMessage` | string | empty for normal chat message or the type of the system message (untranslated)
        `message` | string | Message string with placeholders (see [Rich Object String](https://github.com/nextcloud/server/issues/1706))
        `messageParameters` | array | Message parameters for `message` (see [Rich Object String](https://github.com/nextcloud/server/issues/1706))

## Sending a new chat message

* Method: `POST`
* Endpoint: `/chat/{token}`
* Data:

    field | type | Description
    ------|------|------------
    `message` | string | The message the user wants to say
    `actorDisplayName` | string | Guest display name (ignored for logged in users)

* Response:
    - Header:
        + `201 Created`
        + `400 Bad Request` In case of any other error
        + `404 Not Found` When the conversation could not be found for the participant
        + `413 Payload Too Large` When the message was longer than the allowed limit of 32000 characters (or 1000 until Nextcloud 16.0.1, check the `spreed => config => chat => max-length` capability for the limit)

    - Data:
        The full message array of the new message, as defined in [Receive chat messages of a conversation](#receive-chat-messages-of-a-conversation)

## Get mention autocomplete suggestions

* Method: `GET`
* Endpoint: `/chat/{token}/mentions`
* Data:

    field | type | Description
    ------|------|------------
    `search` | string | Search term for name suggestions (should at least be 1 character)
    `limit` | int | Number of suggestions to receive (20 by default)

* Response:
    - Status code:
        + `200 OK`
        + `404 Not Found` When the conversation could not be found for the participant

    - Data:
        Array of suggestions, each suggestion has at least:

        field | type | Description
        ------|------|------------
        `id` | string | The user id which should be sent as `@<id>` in the message
        `label` | string | The displayname of the user
        `source` | string | The type of the user, currently only `users`
        
## System messages

* `conversation_created` - {actor} created the conversation
* `conversation_renamed` - {actor} renamed the conversation from "foo" to "bar"
* `call_started` - {actor} started a call
* `call_joined` - {actor} joined the call
* `call_left` - {actor} left the call
* `call_ended` - Call with {user1}, {user2}, {user3}, {user4} and {user5} (Duration 30:23)
* `guests_allowed` - {actor} allowed guests in the conversation
* `guests_disallowed` - {actor} disallowed guests in the conversation
* `password_set` - {actor} set a password for the conversation
* `password_removed` - {actor} removed the password for the conversation
* `user_added` - {actor} added {user} to the conversation
* `user_removed` - {actor} removed {user} from the conversation
* `moderator_promoted` - {actor} promoted {user} to moderator
* `moderator_demoted` - {actor} demoted {user} from moderator
* `guest_moderator_promoted` - {actor} promoted {user} to moderator
* `guest_moderator_demoted` - {actor} demoted {user} from moderator
* `read_only_off` - {actor} unlocked the conversation
* `read_only` - {actor} locked the conversation

## Guests
        
        
## Signaling

### Get signaling settings

* Method: `GET`
* Endpoint: `/signaling/settings`
* Data:

    field | type | Description
    ------|------|------------
    `stunservers` | array | STUN servers
    `turnservers` | array | TURN servers
    `server` | string | URL of the external signaling server
    `ticket` | string | Ticket for the external signaling server

    - STUN server
    
       field | type | Description
       ------|------|------------
       `url` | string | STUN server URL

    - TURN server
    
       field | type | Description
       ------|------|------------
       `url` | array | One element array with TURN server URL
       `urls` | array | One element array with TURN server URL
       `username` | string | User name for the TURN server
       `credential` | string | User password for the TURN server

* Response:
    - Header:
        + `200 OK`
        + `404 Not Found`

### External signaling API
See External Signaling API [Draft](https://github.com/nextcloud/spreed/wiki/Signaling-API) in the wiki…
