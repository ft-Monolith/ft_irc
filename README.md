# ft_irc — IRC Server in C++98

> A fully functional IRC server built from scratch in **C++98**, as part of the [42 School](https://42.fr) curriculum.  
> Implements the core IRC protocol using **non-blocking I/O** and `poll()` multiplexing.

---

## 📌 Overview

`ft_irc` is a multi-client IRC server that handles real-time communication between users through channels.  
It was designed to be compliant with standard IRC clients (tested with **irssi**, **HexChat**, **WeeChat**).

The project focuses on:
- **Network programming** with TCP sockets (`AF_INET`, `SOCK_STREAM`)
- **Non-blocking I/O** using `poll()` for concurrent client handling without threads
- **IRC protocol** implementation (RFC 1459)

---

## ⚙️ Architecture

```
ft_irc/
├── srcs/
│   ├── main.cpp              # Entry point
│   ├── messages.hpp          # IRC numeric replies & custom exceptions
│   ├── server/
│   │   ├── Server.hpp        # Core server class
│   │   ├── setup.cpp         # Socket init, command registration
│   │   ├── process.cpp       # Poll loop, client I/O handling
│   │   ├── utils.cpp         # Parsing utilities
│   │   └── signals.cpp       # SIGINT / SIGQUIT graceful shutdown
│   ├── client/
│   │   └── Client.hpp/cpp    # Client state machine (UNREGISTERED → REGISTERED)
│   ├── channel/
│   │   └── Channel.hpp/cpp   # Channel management, modes, operators
│   └── commands/             # One file per IRC command
│       ├── Pass, Nick, User  # Registration flow
│       ├── Join, Part, Quit  # Channel lifecycle
│       ├── Privmsg, Notice   # Messaging
│       ├── Mode, Topic       # Channel configuration
│       ├── Kick, Invite      # Operator actions
│       └── List              # Channel discovery
└── stress_irc.py             # Python async stress-test script
```

---

## 🚀 Build & Run

### Requirements
- `c++` compiler with C++98 support (g++ / clang++)
- POSIX-compatible system (Linux / macOS)

### Compile

```bash
make
```

### Start the server

```bash
./ircserv <port> <password>
# Example:
./ircserv 6667 mypassword
```

### Connect with an IRC client

```bash
# irssi
irssi -c 127.0.0.1 -p 6667 -w mypassword

# netcat (raw testing)
nc 127.0.0.1 6667
```

---

## 📡 Supported IRC Commands

| Command   | Description                                      |
|-----------|--------------------------------------------------|
| `PASS`    | Authenticate with server password                |
| `NICK`    | Set or change nickname                           |
| `USER`    | Register username and real name                  |
| `JOIN`    | Join (or create) a channel                       |
| `PART`    | Leave a channel                                  |
| `PRIVMSG` | Send a message to a user or channel              |
| `NOTICE`  | Send a notice (no auto-reply)                    |
| `MODE`    | Set channel modes (see below)                    |
| `TOPIC`   | Get or set the channel topic                     |
| `KICK`    | Remove a user from a channel (operator only)     |
| `INVITE`  | Invite a user to a channel                       |
| `LIST`    | List available channels                          |
| `QUIT`    | Disconnect from the server                       |

### Channel Modes (`MODE`)

| Flag | Parameter  | Description                                  |
|------|------------|----------------------------------------------|
| `+i` | —          | Invite-only: only invited users can join      |
| `+t` | —          | Only operators can change the topic           |
| `+k` | `<password>` | Require a password to join the channel     |
| `+l` | `<limit>`  | Limit the number of users in the channel      |
| `+o` | `<nick>`   | Grant / revoke operator status                |

---

## 🧪 Stress Test

A Python async stress-test script is included to validate server stability under load.

```bash
# Launch 100 concurrent clients against a running server
python3 stress_irc.py --host 127.0.0.1 --port 6667 --password mypassword --clients 100
```

Output example:
```
=== Bilan du stress-test (100 sessions lancées) ===
✓ réussies           : 100
× connexions refusées : 0
× déconnexions forcées: 0
```

---

## 🛠️ Technical Highlights

- **Single-threaded, event-driven** — all clients handled in one `poll()` loop, no race conditions
- **Non-blocking sockets** — `fcntl(fd, F_SETFL, O_NONBLOCK)` on every connection
- **Input buffering** — partial IRC messages are accumulated per-client until `\r\n` is received
- **Graceful shutdown** — `SIGINT`/`SIGQUIT` cleanly close all connections and free all resources
- **Operator system** — per-channel operator roles with enforcement on restricted commands
- **Exception-based error handling** — `recoverable_error` vs `critical_error` vs `quit_server`
- **Compiled with** `-std=c++98 -Wall -Wextra -Werror`

---

## 📋 Makefile Targets

| Target  | Description                        |
|---------|------------------------------------|
| `make`  | Build the server binary (`ircserv`)|
| `make clean` | Remove object files           |
| `make fclean` | Remove objects + binary      |
| `make re` | Full rebuild                     |
| `make bonus` | Build with bonus features     |
