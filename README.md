# Maverick

Maverick is a multi-platform desktop application that can place bets automatically on multiple online betting platforms using your web browser. Bets are placed in an average of 5 seconds, making it ideal for live matches and real-time betting.

## Supported platforms

- [X] Bet365


## Supported markets

- [X] Result
- [X] Double chance
- [X] Draw no bet
- [X] Goal line
- [X] Goals
- [X] Asian handicap

## Features

- Hands-free bet placement on live matches
  - Error detection on match, market, and bet levels
  - Auto-retry capabilities:
    - Participant suspended
    - Accept odds changed
    - Input quantity failure
    - Click or selection failure
  - Authentication QR code detection

- Hands-free bet closing for active bets
  - Auto-retry on close suspended
  - Minimum cash out threshold
  - Return tracking

- Auto-login (and re-login) with error detection
- Request lifetime history tracking
- Detailed error reporting
- Settings accessible through TOML file


## How to use it?

Maverick is available upon request. Send an email to abel `(at)` jupiter `(dash)` labs `(dot)` tech.

Plans are tailored to your use case, so make sure to include what you need in your email.

You should be up and running in around 2-3 days since your first email.

<details>
  <summary>I need more markets, would you add them?</summary>
  Yes, no problem, should be quick.
</details>

<details>
<summary>What about other betting platforms?</summary>
I'm happy to add other betting platforms upon request. Might incur in extra cost.
</details>

<details>
<summary>Can it place bets on multiple platforms at the same time?</summary>
Yes indeed.
</details>

<details>
<summary>Can it use a Telegram group as feed for its tips?</summary>
Absolutely. There's an open source extension for Maverick available which you can customize for your particular needs and includes Telegram.
</details>

<details>
<summary>Can I run it in my own computer at home or do I need a server?</summary>
You can use both, but at home is always easier.
</details>

<details>
<summary>I need XYZ, can you do it for me?</summary>
Maybe, shoot me an email, let's chat!
</details>

<details>
<summary>Is the full source code available for sale?</summary>
Sure, everyone has a price.
</details>


----


## Configuration

Maverick expects to find a `Config.toml` file in the same directory as the executable. The file is pretty self-explanatory, but here's what to expect from every category:

| Category  | Description                                         |
| --------- | --------------------------------------------------- |
| Server    | Connection URL to the WebSocket (RF6455) server     |
| Browser   | Canonical path to the Chrome browser executable     |
| Platforms | Betting platform credentials (in plain text)        |
| X         | If `post=true` X Developer credentials must be set  |


### Server settings

This is how the server settings should look:

```toml
[server]
addr = "ws://SERVER_HOST:PORT"
max_retries = 10
```

| Property    | Description                                    |
| ----------- | ---------------------------------------------- |
| addr        | The Websocket server address (incl. port)      |
| max_retries | Max. nº of reconnection attempts before abort. |


### Browser settings

This is how the browser settings should look:

```toml
[browser]
bin_path = "/absolute/path/to/chrome"
window_size = [1500, 900]
```

| Property       | Description                                          |
| -------------- | ---------------------------------------------------- |
| bin_path       | The absolute path of the Chrome binary.              |
| window_size*   | An array specifying window width and height.         |
| _window_pos_*  | _(Optional)_ Window left and top offset coordinates. |

*_Window size and position settings may not as expected work on macOS._


### Platform settings

This is how a platform configuration should look:

```toml
[platforms.bet365]
host = "www.bet365.es"
username = "theBetGod"
password = "iLoveMe$$i!"
language = "es"
```

| Property | Description                              |
| -------- | ---------------------------------------- |
| host     | The root domain/host for the platform    |
| username | The username of the account used to bet  |
| password | The account password (in plain text)     |
| language | The language of the account used to bet  |


## Running Mavierick

Maverick is contained in a single binary and can be excuted normally

### macOS
```bash
$ ./maverick
```

### Windows
```bash
maverick.exe
```

## Interfacing with Maverick

On startup, Maverick will automatically connect to the WebSocket server defined in the configuration file. On any disconnect, it will attempt to reconnect `max_retries` times. If a connection can't be established, Maverick will gracefully shut down.

Maverick supports 3 commands from the upstream:

| Command      | Description                           |
| ------------ | ------------------------------------- |
| SessionStart | Starts a new session.                 |
| PlaceBet     | Attempts to place a bet               |
| CloseBet     | Attempts to close an open/active bet  |


### SessionStart

SessionStart will effectively start a new session. This means:
1. Dump full bet history for current session into a session log file.
2. Clear bet history and start a new, fresh betting session.
3. Reset equity for this session with current account balance.

#### Example:
```json
"SessionStart"
```

### PlaceBet

PlaceBet will attempt to place a bet in the given match. The main parameters here are:

- `id`: A UUID v4 that will uniquely identify this bet request.
- `match`: The details to identify the live match.
- `bet`: The actual market and participant we're betting on.
- `odds`: The odds we are looking for.
- `stake`: The amount of capital we want to deploy.

#### Identifying a match

There's currently two ways of specifying a match:

##### Using the platform url:
```json
{
  "Url": "https://www.bet365.es/#/IP/EV15970168472C1"
}
```

##### Using the team names:
```json
{
  "Teams": ["Roma", "Lazio"]
}
```


#### Bets

For every supported market, there are multiple ways of specifying the participant:

#### Result

##### Using participant index:
```json
{
  "Result":{
    "Index": 2
  }
}
```

##### Using participant name:
```json
{
  "Result":{
    "Name": "FC Andorra"
  }
}
```

#### Double Chance

##### Using participant index:
```json
{
  "DoubleChance":{
    "Index": 0
  }
}
```

##### Using participant name:
```json
{
  "DoubleChance":{
    "Name": "Oviedo o Empate"
  }
}
```

#### Draw No Bet

##### Using participant index:
```json
{
  "DrawNoBet":{
    "Index": 0
  }
}
```

##### Using participant name:
```json
{
  "DrawNoBet":{
    "Name": "Sevilla"
  }
}
```

#### Goal Line

##### Over:
```json
{
  "GoalLine":{
    "score": [1,0],
    "goals":{
      "Over": "1.5"
    }
  }
}
```

##### Under:
```json
{
  "GoalLine":{
    "score": [0,0],
    "goals":{
      "Under": "0.5,1.0"
    }
  }
}
```

#### Asian Handicap

##### Using participant index and single hcp:
```json
{
  "AsianHcp":{
    "participant":{
      "Index":0
    },
    "score":[1,0],
    "line":[1.5]
  }
}
```

##### Using participant name and spread hcp:
```json
{
  "AsianHcp":{
    "participant":{
      "Name":"Roma"
    },
    "score":[1,0],
    "line":[-0.5, -1]
  }
}
```

#### Goals

##### Over:
```json
{
  "Goals":{
    "Over": "1.5"
  }
}
```

##### Under:
```json
{
  "Goals":{
    "Under": "4.5"
  }
}
```

#### Full Example: A complete bet request
```json
{
  "PlaceBet":{
    "id": "60f293c2-3ce1-49fb-a309-1c75520fc1d2",
    "match":{
      "Url": "https://www.bet365.es/#/IP/EV15970168472C1"
    },
    "bet":{
      "Result":{
        "Index": 2
      }
    },
    "odds":{
      "base": 1.4
    },
    "stake": 0.05
  }
}
```

### Odds

The odds have a default threshold of ±10% from the `base` value. Optionally, this threshold can be adjusted by adding the `tolerance` parameter as shown below:

```json
{
  "base": 1.4,
  "tolerance": 0.25
}
```

The `tolerance` value is normalized, so only values in the range `0..1` should be used. In the example, `0.25` would allow for a 25% deviation to either side from the base `1.4` value, so effectively the bet would be placed if the actual value offered by the platform is within the range `1.05..1.75`.

### Stake

The nominal cash amount of the bet is effectively derived from the `stake` value, which again should always be in the range `0..1`.

Assuming `B` is the session starting balance, the bet amount will be calculated as follows:

```
Bet Amount = B × stake
```

So, as an example, if the session starting balance `B` is 100€ and the `stake` for a given bet request is `0.05`, Maverick will attempt to place the bet using 5€ of capital.


### CloseBet

CloseBet will attempt to close an open/active bet. The request is pretty straightfoward:
- `bet_req_id` corresponds to the original bet request id used in `PlaceBet` when the bet was placed.
- `min_cashout` is the minimum **percent** cashout value relative to the wager placed for Maverick to close the bet.

#### Example:
```json
{
  "CloseBet":{
    "bet_req_id":"dfc28c57-1ee8-478f-b863-0dd665049570",
    "min_cashout":1.60
  }
}
```


## Maverick responses

Occasionally, Maverick will send data back to the Websocket server. These are the messages that can be expected.

### Session started

Every time a new platform is logged in, a `SessionData` message will be sent back to the server containing:
- Starting time of the current session
- Starting balance of all currently logged in platforms

Here's an example:

```json
{
  "date":1702985129669,
  "start_balance":{
    "bet365.es": 995.24,
    "bwin.es": 52.98
  }
}
```

### Bet Reports

When either a `PlaceBet` or `CloseBet` is completed, Maverick will always send a full `BetRequest` object back to the server with all the details and history of everything that happened during that request, including (if applies) the close.

Here's a complete example of what it might look like:

```json
{
  "id":"e3df861b-140f-43ee-bb2d-f95484a8659e",
  "url":"https://www.bet365.es/#/IP/EV15983185942C1",
  "bet":{
    "AsianHcp":{
      "team":{
        "Index":0
      },
      "score":[0,0],
      "line":[0.0,-0.5]
    }
  },
  "odds":{
    "base":3.0,
    "tolerance":0.1
  },
  "tf":null,
  "stake":0.05,
  "amt":4.89,
  "hist":[
    {
      "timestamp":1702985124368,
      "status":"Received"
    },
    {
      "timestamp":1702985129661,
      "status":"Ready"
    },
    {
      "timestamp":1702985129669,
      "status":{
        "Error":{
          "OddsChanged":2.075
        }
      }
    }
  ],
  "min_cashout":0.0,
  "match_info":null
}
```

#### Errors

The following is a comprehensive list of all errors that can occur while executing a `BetRequest`.

| Error code                  | Description                                                |
| --------------------------- | ---------------------------------------------------------- |
| MarketSuspended             | Market was suspended at the time of attempting the bet     |
| OddsChanged(f32)            | Odds changed and are out of the specified tolerance factor |
| NotLoggedIn                 | User is not logged in                                      |
| MatchNotFound               | Match not found (probably finished)                        |
| MarketNotSupported          | Market is not supported                                    |
| MarketNotAvailable          | Market was not available at the time of attempting the bet |
| ParticipantNotFound         | Participant was not found                                  |
| NoOpenBets                  | No bets are open for the specified match                   |
| BetNotFound                 | The requested bet (to close) was not found                 |
| CloseSuspended              | Closing was suspended when attempting to close this bet    |
| CloseNotAvailable           | Closing is not available for the requested bet             |
| BelowMinCashout(f32)        | Bet return is below min cashout % threshold                |
| Timeout(String)             | Took to long to perform a task                             |
| BetMalformed(String)        | An error in the bet request's betting amount               |
| AuthQRCodeRequested         | Platform requested QR authentication by a user             |
| PlatformErrorMsg(String)    | Platform error message appeared when placing a bet         |
| Other(String)               | Other errors (includes details as a String)                |
| Unknown                     | Unknown error                                              |
