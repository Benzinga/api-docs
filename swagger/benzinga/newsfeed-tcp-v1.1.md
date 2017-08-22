# TCP Newsfeed v1.1

## Line Endings

All lines endings will be in the expected format:

```
=BZEOT\r\n
```

## Commands

All commands that return or send data will include a colon (:) and possible whitespace ( ) between the command and the data.

**Note:** A single whitespace was required and will now be deprecated.

## Connection

All clients are given a username and key to connect with. Only one connection is allowed from a username/key at any given time.

Once connected, the server will reply with **READY**

on **READY**, send an **AUTH** token in the form:

```
AUTH:JSON-Object {

  username: username

  key: 1234567890

}
```

The server will reply **CONNECTED**, if the connection is valid, otherwise disconnect.

**Example:**

```
SERVER> READY=BZEOT\r\n

CLIENT> AUTH: {"username": "username", "key": "1234567890"}=BZEOT\r\n

SERVER> CONNECTED=BZEOT\r\n
```


### No Authentication Received

Clients will receive the following disconnect message if no authentication message is received:

```
Goodbye=BZEOT\r\n
```


### Failed Authentication

Clients failing authentication will receive the following message(s):

```
INVALID KEY=BZEOT\r\n
```
```
INVALID KEY FORMAT=BZEOT\r\n
```

### Multiple Connections

If multiple connections are needed in your environment, please contact us.

Multiple connections with the same Username/Key will result in a disconnect:

```
DUPLICATE CONNECTION=BZEOT\r\n
```

## Keep Alive

### Client Initiated

After successful connection, a client should send a keep alive by sending PING: along with the pingTime in JSON format. The pingTime will be returned in the response by the server, along with the server time in UTC. The pingTime may be in any string or numeric format and in only used as the response token.

**Example:**

```
CLIENT> PING: {"pingTime": "VALUE"}=BZEOT\r\n

SERVER> PONG: {"serverTime":"Thu Sep 20 2012 01:02:51 GMT+0000 (UTC)", "pingTime":"VALUE"}=BZEOT\r\n
```

## Data

Once connected, article Data will be streamed in JSON format. The following data will be available:

**Example:**

```
STREAM: {
  id: 1234567,                                          // Unique ID of Story
  title: 'TITLE OF STORY',                              // TITLE of Article
  body: 'BODY OF STORY',                                // Body of Article
  status: 'Published',                                  // Published or Removed
  published: 'Thu Aug 09 2012 19:41:09 GMT+0000 (UTC)', // Published Date, GMT
  updated: 'Thu Jan 01 1970 00:00:00 GMT+0000 (UTC)',   // Revision Date, GMT
  link: 'http://www.benzinga.com/link/to/story',        // Link to Article
  channel: [                                            // Array of Channels
    'CHANNEL',
    'CHANNELX'
  ],
  tickers: ['A', 'B', 'C']                              // Array of Tickers
}=BZEOT\r\n
```

**Example (with extended tickers):**

```
STREAM: {
  id: 1234567,	                                        // Unique ID of Story
  title: 'TITLE OF STORY',	                            // Title of Article
  body: 'BODY OF STORY',                                // Body of Article
  status: ‘Published’,	                                // Published or Removed
  published: 'Thu Aug 09 2012 19:41:09 GMT+0000 (UTC)', // Published Date, GMT
  updated: 'Thu Jan 01 1970 00:00:00 GMT+0000 (UTC)',	  // Revision Date, GMT
  link: ‘http://www.benzinga.com/link/to/story’,	      // Link to the Article
  channels: [	                                          // Array of Channels
    'CHANNEL' ,
    ‘CHANNELX’
  ],
  tickers: [                                            // Array of Tickers
    {	
      name: 'F',	                                      // Ticker Symbol
      primary: 1,                                       // Denotes Primary Status (0, 1)
      sentiment: false	                                // Sentiment (-4 to 4) if available
    }
  ]
}=BZEOT\r\n
```

### Data Fields

```
definitions:

  id:
    type: integer
    definition: Primary Key and Unique ID of the Story

  title:
    type: string
    max-length: 255 chars
    definition: Title of the Article

  body:
    type: string
    max-length: Unlimited / Varying
    definition: Body of the Article. The length may be limited, per client setup. Body format may be in HTML or RichText per client setup.

  published:
    type: string
    definition: This is the published date of the article. Date will be in GMT. Example: 'Thu Aug 09 2012 19:41:09 GMT+0000 (UTC)'

  updated:
    type: string
    definition: This the updated timestamp of the article. This may be updated for a number of reasons that may or may not be apparent to the client. Stories will be pushed on all updates. Date will be in GMT.
    example: 'Thu Jan 01 1970 00:00:00 GMT+0000 (UTC)'

  status:
    type: string
    definition: This is the status of an article. A status of either “Published” or “Removed” will be included with the article. Article updates would only be indicated through a pushed and a client checking the difference in updated times.
    example: “Published” or “Removed”

  link:
    type: string
    definition: Link to publicly available story on Benzinga.com. This field will indicate NULL if a link is not available.

  channels:
    type: array
    definition: The Benzinga Channels or categories an article appears in. This array may be blank if no channels exist.
    example: [‘News’ , ‘Markets’ ]

  tickers:
    type: array
    definition: Array of symbols
    items:
      type: string
      description: Ticker symbol
    example: >- 
      tickers: [‘F’, ‘GM’]
       
  tickers(extended):
    type: array
    description: Array of symbol objects
    items:
      type: object
      properties:
        name:
          type: string
          description: The actual ticker symbol
        primary:
          type: integer
          format: boolean
          description: 1 if primary 0 if not
        sentiment:
          type: integer
          description: Sentiment for this symbol as a result of this story (if available). -4 - 4 scale. FALSE if not available.
      example: >-
        tickers: [ { name: 'F', primary: 1, sentiment: 1 } ] 

```
