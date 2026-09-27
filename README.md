# Meshtastic Trivia Bot

**Meshtastic Trivia Bot** is a daily quiz bot for the **Meshtastic Community's Bulgaria channel**.

It connects directly to a Meshtastic node over **TCP or serial**, posts one random trivia question every day, accepts answers directly from the channel, keeps player statistics, and maintains a monthly leaderboard.

## Features

* 🧠 One random trivia question every day at **20:00**
* ⏱️ Players have **5 minutes** to answer
* 🥇 Only the **first correct answer** counts
* 🤫 Wrong answers are ignored silently
* 📊 Persistent player statistics
* 🏆 Top-10 leaderboard
* 📅 Automatic monthly scoreboard reset
* 📬 Private replies for `TOP`, `STATS` and `HELP`
* 🔤 Handles visually identical Latin/Cyrillic characters when matching answers
* 📡 Supports both TCP and serial Meshtastic connections
* 📦 Automatically splits messages that exceed the Meshtastic payload size
* 🔄 Automatically reconnects if the Meshtastic connection is lost
* 💾 Statistics survive bot restarts

## Daily Game

Every day at **20:00 local time**, the bot broadcasts a random question to the Bulgaria channel.

Example:

```text
🧠 Коя е столицата на България? (5м)
```

The first participant to send the correct answer wins the round.

The bot then announces the winner:

```text
✅ !12345678 отговори правилно на въпроса на деня за 12.4с!
Браво! Следващ въпрос утре в 20:00.
```

If nobody answers correctly within the five-minute window, the question simply expires without an additional channel announcement.

## Commands

Commands are sent directly to the bot through the Meshtastic channel.

| Command | Description                                  |
| ------- | -------------------------------------------- |
| `TOP`   | Returns the current top-10 leaderboard by DM |
| `STATS` | Returns your personal statistics by DM       |
| `L`     | Alias for `STATS`                            |
| `HELP`  | Returns the available commands by DM         |

The bot does not broadcast leaderboard or personal statistics to the entire channel.

## Scoring

Each player is identified by their Meshtastic node ID.

For every correct answer the bot records:

* Number of wins
* Total response time
* Fastest response time
* First date the player was seen
* Date/time of the most recent win

The leaderboard is ordered by:

1. Number of wins
2. Average response time

The leaderboard displays the top 10 players.

Example:

```text
🏆 Топ класация 🏆
🥇 !12345678: 12 победи (avg 18.4s)
🥈 !87654321: 9 победи (avg 21.7s)
🥉 !abcdef01: 7 победи (avg 16.2s)
4. !11223344: 6 победи (avg 25.1s)
```

## Monthly Scoreboard

The monthly scoreboard is enabled by default.

At the beginning of a new month:

* The previous month's winner is announced.
* Monthly wins are reset.
* Total response time is reset.
* Fastest response time is reset.
* Player identity and history fields are retained.

Statistics are stored locally in:

```text
trivia_stats.json
trivia_meta.json
```

The metadata file allows the bot to remember the last monthly reset even after a restart.

## Question Database

Questions are currently built directly into the Python source.

The database contains categories including:

* 🔬 Science & Nature
* 📜 History
* 🌍 Geography
* 🧪 Science, formulas and units
* ➗ Arithmetic & Mathematics
* 📚 General Knowledge
* 📻 Radio & Networking

The questions are primarily in Bulgarian and include topics ranging from general knowledge to radio, networking and Meshtastic-related technology.

## Answer Matching

The bot handles a common problem with Bulgarian mobile keyboards: mixing visually identical Latin and Cyrillic characters.

For example, Latin:

```text
a
```

and Cyrillic:

```text
а
```

are different Unicode characters even though they look identical.

The bot uses the `confusables` Python package to detect visually confusable answers instead of relying exclusively on a simple string comparison.

Answers are also normalized by:

* Removing surrounding whitespace
* Removing surrounding punctuation
* Converting to lowercase

## Message Size Handling

Meshtastic transmission limits are measured in **bytes**, not characters.

This is particularly important for Bulgarian text because Cyrillic characters use multiple UTF-8 bytes.

The bot therefore automatically splits long messages into smaller chunks.

For example:

```text
[1/3] 🏆 Топ класация...
[2/3] ...
[3/3] ...
```

Direct messages use a shorter delay between chunks, while channel broadcasts use a longer delay to avoid overwhelming multi-hop radio traffic.

Current defaults:

```text
DM maximum:              180 bytes
DM chunk delay:            5 seconds
Broadcast chunk delay:    20 seconds
```

## Meshtastic Connection

The bot supports two connection methods.

### TCP

```python
CONNECTION_TYPE = "tcp"
NODE_IP = "192.168.78.58"
```

### Serial

```python
CONNECTION_TYPE = "serial"
SERIAL_PORT = "/dev/ttyUSB0"
```

Only one connection type is active at a time.

The primary channel is configured with:

```python
CHANNEL_INDEX = 0
```

## Configuration

The main configuration is located near the beginning of the Python file.

```python
CONNECTION_TYPE = "tcp"
NODE_IP = "192.168.78.58"
SERIAL_PORT = "/dev/ttyUSB0"
CHANNEL_INDEX = 0

STATS_FILE = "trivia_stats.json"
META_FILE = "trivia_meta.json"

MONTHLY_RESET_ENABLED = True

QUESTION_HOUR = 20
QUESTION_MINUTE = 0

ANSWER_WINDOW = 3600
```

Adjust these values to match the Meshtastic node and the desired schedule.

## Requirements

Install the Python dependencies with:

```bash
pip install -r requirements.txt
```

`requirements.txt`:

```text
meshtastic
pyserial
pypubsub
confusables
```

Python standard-library modules used by the bot do not need to be installed separately.

## Running

Make the script executable:

```bash
chmod +x trivia_bot.py
```

Run it:

```bash
./trivia_bot.py
```

or:

```bash
python3 trivia_bot.py
```

On startup the bot reports:

* Connection method
* Meshtastic node address
* Number of questions
* Daily question time
* Answer window
* Connected node ID

Example:

```text
Initializing TriviaBot v3.0 (daily question mode)...
Connecting via TCP to 192.168.78.58...
Connection established
Subscribed to message events
Scheduler running — daily question at 20:00
Running | TCP @ 192.168.78.58 | 300 questions | Daily at 20:00 | Answer window: 5 min
Node ID: 123456789
Ctrl+C to quit
```

## Persistence

The bot keeps its data in JSON files in the working directory.

### `trivia_stats.json`

Contains player statistics and is updated after every correct answer.

### `trivia_meta.json`

Stores the month of the last scoreboard reset.

The bot also uses a temporary backup file while updating the statistics database to reduce the risk of losing the existing statistics if a write fails.

## Reconnection

The bot continuously monitors the Meshtastic connection.

If the connection is lost, it attempts to reconnect automatically every 10 seconds.

This allows the trivia service to continue operating without requiring manual intervention after a temporary node or network failure.

## Community

This bot is intended for the **Meshtastic Community's Bulgaria channel**, providing a small daily activity for members of the Bulgarian Meshtastic community.

It is a community project built around Meshtastic and is not part of the official Meshtastic project.

## License

Add the project's license here if/when one is selected.
