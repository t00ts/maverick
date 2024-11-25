# Maverick

Maverick is an app that can place bets automatically on Bet365 leveraging your existing web browser. It runs on Windows and macOS (Linux coming soon) and simulates human behaviour to avoid being detected. Bets are placed in an average of 5 seconds, making it ideal for live matches and real-time betting.

Maverick itself is just your executor arm for unattended betting. You can think of it as an API for Bet365. Maverick does not include a betting strategy nor will perform any bets by itself. It's up to the user to send instructions to Maverick for them to be executed.

Instructions are sent via Websockets. The user is responsible for spinning up a Websocket server, through which it can send and receive commands from Maverick. An example server that is commonly used to start off can be found [here](https://github.com/t00ts/maverick-server). You can connect as many Maverick clients to your server as you want.

Basic programming knowledge is strongly recommended. This is not a plug-and-play solution.

## How to use it?

Maverick is available upon request:
- Email: abel `(at)` akila `(dot)` tech
- Telegram:  `(at)` abel `(underscore)` maverick.

You should be up and running in around 2-3 days since your first contact.

<details>
  <summary>I need more markets, would you add them?</summary>
  Yes, no problem, should be quick.
</details>

<details>
<summary>What about other betting platforms?</summary>
We can add other betting platforms for those willing to sponsor their development.
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

## Supported bet types

- [X] Single bets
- [X] Multi-legged bets (a.k.a. "parlay" or "accumulator")


## Supported markets

- [X] Result
- [X] Double chance
- [X] Draw no bet
- [X] Goal line
- [X] Goals
- [X] Asian handicap
- [X] Score

## Features

- Hands-free bet placement on live matches
  - Error detection on match, market, and bet levels
  - Auto-retry capabilities:
    - Participant suspended
    - Accept odds changed
    - Input quantity failure
    - Click or selection failure
  - Authentication QR code detection
  - Authentication QR code solving

- Hands-free bet closing for active bets
  - Auto-retry on close suspended
  - Minimum cash out threshold
  - Return tracking

- Auto-login (and re-login) with error detection
- Request lifetime history tracking
- Detailed error reporting
- Bet queuing
- Settings accessible through TOML file

## How to use

### 1. Configuration

Maverick expects to find a `Config.toml` file in the same directory as the executable. The file is pretty self-explanatory, but here's what to expect from every category:

| Category  | Description                                                                               |
| --------- | ----------------------------------------------------------------------------------------- |
| Server    | Connection URL to the WebSocket (RF6455) server that will send the commands to Maverick   |
| Browser   | Canonical path to the Chrome browser executable                                           |
| Platforms | Betting platform credentials (in plain text) and additional features to be enabled        |
| X         | Developer credentials to post your bets on 𝕏                                              |


#### Server settings

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


#### Browser settings

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


#### Platform settings

This is how a platform configuration should look:

```toml
[platforms.bet365]
host = "www.bet365.com"
username = "theBetGod"
password = "iLoveMe$$i!"
language = "en"
features = []
```

| Property | Description                              |
| -------- | ---------------------------------------- |
| host     | The root domain/host for the platform    |
| username | The username of the account used to bet  |
| password | The account password (in plain text)     |
| language | The language of the account used to bet  |
| features | Additional features to be enabled        |


### 2. Running Mavierick

Maverick is contained in a single binary and can be excuted using the command line.

**Note:** It is strongly recommended to run Maverick using a proper terminal/command line interface. Especially on Windows, the default console is known to cause issues when dealing with certain characters and colors.

#### macOS
```bash
$ ./maverick
```

#### Windows
```bash
maverick.exe
```

On startup, Maverick will automatically connect to the WebSocket server defined in the configuration file. On any disconnect, it will attempt to reconnect `max_retries` times. If a connection can't be established, Maverick will gracefully shut down.


### 3. Interfacing with Maverick

> 💡 We strongly encourage all new users to play around with the `betreq` CLI utility included in your Maverick bundle in order for you to understand how to specify your bets and get familiar with the API syntax.
> 
> On Windows: `betreq.exe`
> 
> On macOS/Linux: `./betreq`


Maverick supports 3 commands from the upstream:

| Command         | Description                           |
| --------------- | ------------------------------------- |
| `session_start` | Starts a new session.                 |
| `place_bet`     | Attempts to place a bet               |
| `close_bet`     | Attempts to close an open/active bet  |


#### Session Start

`session_start` will effectively start a new session. This means:
1. Dump full bet history for current session into a session log file.
2. Clear bet history and start a new, fresh betting session.
3. Reset equity for this session with current account balance.

#### Example:
```json
"session_start"
```

#### Place Bet

`place_bet` will attempt to place a bet in the given match. `place_bet` contains a `BetRequest` object,
for which the main parameters are:

- `id`: A UUID that will uniquely identify this bet request on your system.
- `host`: The root domain for the betting platform (platform `host` value from `Config.toml`)
- `bet`: The bet(s) to be placed, which contains:
  - `match`: The details to identify the live match _(see below for more details)_.
  - `market`: The actual market and participant we're betting on _(see below for more details)_.
  - `odds`: The minimum odds we are looking for to place the bet.
  - `tf`: The time frame for the bet _(e.g. `FirstHalf`, `SecondHalf`, `FullTime`)_.
  - `stake`: The amount of capital to be deployed on this bet.

#### Identifying a match

There's currently two ways of specifying a match:

##### Using the platform url:
```json
"match": "https://www.bet365.com/#/IP/EV00000012345678"
```

##### Using the team names:
```json
"match": ["Roma", "Lazio"]
```

> _Note: Combined bets only support url-identified matches._

#### Bets

Some examples of actual bets:

##### Result

```json
"bet": {
  "market": {
    "result": "Ajax FC"   // Uses name to specify participant
  },
  "match": "https://www.bet365.com/#/IP/EV00000012345678",
  "odds": 3.2,
  "tf": "FullTime"
}
```

##### Double Chance

```json
"bet": {
  "market": {
    "double_chance": 1  // Uses index to specify participant
  },
  "match": "https://www.bet365.com/#/IP/EV00000012345678",
  "odds": 2.1,
  "tf": "FullTime"
}
```

##### Draw No Bet

```json
"bet": {
  "market": {
    "draw_no_bet": "Sevilla"
  },
  "match": "https://www.bet365.com/#/IP/EV00000012345678",
  "odds": 1.8,
  "tf": "FullTime"
}
```

##### Goal Line (single)

```json
"bet": {
  "market": {
    "goal_line": {
      "over": [3.0],
      "score": [1, 2]
    }
  },
  "match": "https://www.bet365.com/#/IP/EV00000012345678",
  "odds": 3.45,
  "tf": "FirstHalf"
}
```

##### Goal Line (split)

```json
"bet": {
  "market": {
    "goal_line": {
      "score": [0,1],
      "under": [2.0, 2.5]
    }
  },
  "match": "https://www.bet365.com/#/IP/EV00000012345678",
  "odds": 2.5,
  "tf": "FullTime"
}
```

##### Goals

```json
"bet": {
  "market": {
    "goals": {
      "over": 4.0
    }
  },
  "match": "https://www.bet365.com/#/IP/EV00000012345678",
  "odds": 2.83,
  "tf": "FirstHalf"
}
```

##### Asian Handicap (single line)

```json
"bet": {
  "market": {
    "asian_hcp": {
      "line": [1.5],
      "score": [0, 2],
      "team": "Manchester City"
    }
  },
  "match": "https://www.bet365.com/#/IP/EV00000012345678",
  "odds": 4.15,
  "tf": "FullTime"
}
```

##### Asian Handicap (spread)

```json
"bet": {
  "market": {
    "asian_hcp": {
      "line": [-0.5, -1],
      "score": [0, 0],
      "team": "Udinese FC"
    }
  },
  "match": "https://www.bet365.com/#/IP/EV00000012345678",
  "odds": 2.5,
  "tf": "FullTime"
}
```

###### Catching fast-changing handicaps

You can specify a tolerance factor for fast-changing handicaps. In the following example, all line values have a ±0.25 tolerance factor, which you can adapt to your needs.

```json
"line":[[3.5],0.25]
```

```json
"line":[[0.0, 0.5],0.25]
```

##### Score

Score is always a 2-element array, where the first element is the home team score and the second is the away team score.

```json
"bet": {
  "market": {
    "score": [3, 4]
  },
  "match": "https://www.bet365.com/#/IP/EV00000012345678",
  "odds": 11.5,
  "tf": "FirstHalf"
}
```


#### Full Example: A complete multi-legged bet request
```json
{
  "place_bet": {
    "bet": [
      {
        "market": {
          "result": "Leicester City"
        },
        "match": "https://www.bet365.com/#/IP/EV00000012345678",
        "odds": 15.3,
        "tf": "FullTime"
      },
      {
        "market": {
          "goals": {
            "over": 3.0
          }
        },
        "match": "https://www.bet365.com/#/IP/EV00000012345678",
        "odds": 4.1,
        "tf": "FirstHalf"
      },
      {
        "market": {
          "score": [2, 1]
        },
        "match": "https://www.bet365.com/#/IP/EV00000012345678",
        "odds": 3.85,
        "tf": "FullTime"
      }
    ],
    "host": "www.bet365.com",
    "id": "7e81d4da-3440-44d8-ad29-edc84fb73157",
    "stake": 25
  }
}
```

#### Odds

The minimum odds required to place the bet are set in `odds`. If the platform offers anything above, the bet will be placed.


### Stake

The nominal cash amount to use for the bet, in the account currency.


### CloseBet

CloseBet will attempt to close an open/active bet. The request is pretty straightfoward:
- `bet_req_id` corresponds to the original bet request id used in `PlaceBet` when the bet was placed.
- `min_cashout` is the minimum **percent** cashout value relative to the wager placed for Maverick to close the bet.

#### Example:
```json
{
  "close_bet": {
    "bet_req_id": "7e81d4da-3440-44d8-ad29-edc84fb73157",
    "min_cashout": 1.6
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
    "bet365.com": 995.24,
    "bwin.es": 52.98
  }
}
```

### Bet Reports

When either a `PlaceBet` or `CloseBet` is completed, Maverick will always send a full `BetRequest` object back to the server with all the details and history of everything that happened during that request, including (if applies) the close.

Here's a complete example of what it might look like:

```json
{
  "id":"0dd700b3-0a9c-452e-b3f6-c0228f9492aa",
  "host":"www.bet365.com",
  "bet":[
    {
      "match":{
        "Url":"https://www.bet365.com/#/AC/B1/C1/D8/E161801311/F3/"
      },
      "market":{
        "Goals":{
          "Over":2.5
        }
      },
      "odds":1.66,,
      "tf":"FullTime"
    },
    {
      "match":{
        "Url":"https://www.bet365.com/#/AC/B1/C1/D8/E161190320/F3/"
      },
      "market":{
        "Score":[1, 2]
      },
      "odds":23.0,
      "tf":"FullTime"
    },
    {
      "match":{
        "Url":"https://www.bet365.com/#/AC/B1/C1/D8/E161190309/F3/I1/"
      },
      "market":{
        "Score":[3, 2]
      },
      "odds":34.0,
      "tf":"FullTime"
    },
    {
      "match":{
        "Url":"https://www.bet365.com/#/AC/B1/C1/D8/E161190315/F3/"
      },
      "market":{
        "Score":[2, 2]
      },
      "odds":40.0,
      "tf":"FirstHalf"
    },
    {
      "match":{
        "Url":"https://www.bet365.com/#/AC/B1/C1/D8/E161190989/F3/"
      },
      "market":{
        "Score":[0, 0]
      },
      "odds":2.6,
      "tf":"FirstHalf"
    }
  ],
  "stake":5,
  "hist":[
    {
      "timestamp":1727283011713,
      "status":"Received"
    },
    {
      "timestamp":1727283012883,
      "status":"Ready"
    },
    {
      "timestamp":1727283016891,
      "status":{
        "Loaded":"https://www.bet365.com/#/AC/B1/C1/D8/E161801311/F3/"
      }
    },
    {
      "timestamp":1727283023366,
      "status":{
        "Loaded":"https://www.bet365.com/#/AC/B1/C1/D8/E161190320/F3/"
      }
    },
    {
      "timestamp":1727283030439,
      "status":{
        "Loaded":"https://www.bet365.com/#/AC/B1/C1/D8/E161190309/F3/I1/"
      }
    },
    {
      "timestamp":1727283031093,
      "status":{
        "Error":{
          "OddsChanged":41.0
        }
      }
    }
  ]
}
```

When the bet is actually placed, the last status update will look like this:
```json
{
  "timestamp":1727176294823,
  "status":{
    "Placed":{
      "ref":"BF7709201861A",
      "amt":5.0,
      "odds":123.45,
      "acct_balance":94.30
    }
  }
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
| MatchSuspended              | Match was suspended at the time of attempting the bet      |
| MarketNotSupported          | Market is not supported                                    |
| MarketNotAvailable          | Market was not available at the time of attempting the bet |
| ParticipantNotFound         | Participant was not found                                  |
| NoOpenBets                  | No bets are open for the specified match                   |
| BetNotFound                 | The requested bet (to close) was not found                 |
| CloseSuspended              | Closing was suspended when attempting to close this bet    |
| CloseNotAvailable           | Closing is not available for the requested bet             |
| BelowMinCashout(f32)        | Bet return is below min cashout % threshold                |
| Timeout(String)             | Took to long to perform a task                             |
| BetMalformed(String)        | An error in the bet request's betting amount               |
| AuthQRCodeRequested(String) | Platform requested QR authentication by a user             |
| QRSolverService(String)     | Errors originated from the QR Solver service               |
| PlatformErrorMsg(String)    | Platform error message appeared when placing a bet         |
| License(String)             | License error (includes details as a String)               |
| Other(String)               | Other errors (includes details as a String)                |
| Unknown                     | Unknown error                                              |


## Runtime requirements

Maverick is expected to run on a dedicated environment. It's not meant to be used in hostile environments nor anywhere where a human is interfering with its operation.

- Chrome needs to be in the main display of the computer running Maverick.
- Chrome must remain in foreground at all times.
- Chrome must be exclusively used by Maverick.
- Betting platforms have been manually granted all the permissions they require.
- Betting accounts used have been manually authorized by a human.
