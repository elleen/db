# db

Small bash tools for saving MySQL connections and opening them with one menu pick.

| Command     | What it does                                                  |
| ----------- | ------------------------------------------------------------- |
| `db`        | Pick a saved connection and open an interactive `mysql` shell |
| `db-add`    | Save (or replace) a connection                                |
| `db-remove` | Delete a saved connection                                     |

## Requirements

- bash, macOS or Linux
- `mysql` client on your `PATH`
- `lsof` (only for connections that use a tunnel)

## Install

Clone the repo and put it on your `PATH`:

```sh
git clone git@github.com:elleen/db.git ~/bin
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
```

Make the scripts executable (git normally preserves this, but run it if you get `permission denied`):

```sh
chmod +x ~/bin/db ~/bin/db-add ~/bin/db-remove
```

Then open a new shell, or run `source ~/.zshrc`, so the `PATH` change takes effect.

## Usage

### Add a connection

`db-add` takes the same flags as `mysql`, then asks for an alias:

```sh
db-add mysql -h db.example.com -P 3306 -u myuser -p mydb
Alias: example-prod
```

| Flag                   | Notes                                                       |
| ---------------------- | ----------------------------------------------------------- |
| `-h`, `--host`         | Required                                                    |
| `-u`, `--user`         | Required                                                    |
| `-P`, `--port`         | Defaults to `3306`                                          |
| `-p`                   | Prompts for the password (recommended)                      |
| `-pPASSWORD`           | Passes the password inline (lands in your shell history)    |
| `-D`, `--database`     | Default schema; a bare trailing argument works too          |
| `--tunnel=CMD`         | Command to open a tunnel first, see [Tunnels](#tunnels)     |

Adding an alias that already exists replaces it.

### Connect

```sh
db
```

```
Which database?
  1) example-prod         db.example.com:3306
  2) example-local        127.0.0.1:3307
Choose [1-2]: 1
Which table/schema? [mydb]:
Connecting to db.example.com:3306/mydb as myuser...
```

Press Enter at the schema prompt to use the saved default.

### Remove a connection

```sh
db-remove              # pick from a numbered list
db-remove example-prod # or name it directly
```

## Tunnels

If a connection has a tunnel command, `db` runs it in the background before connecting and stops it when you exit `mysql`. Use it for hosts that are only reachable through a bastion, for example:

```sh
db-add mysql -h 127.0.0.1 -P 3307 -u myuser -p mydb \
  --tunnel='ssh -N -L 3307:db.internal:3306 bastion'
```

- The host and port are what `mysql` connects to, so point them at the local end of the tunnel.
- `db` kills anything already listening on that local port before starting the tunnel.
- It waits up to 30 seconds for the port to open. If the tunnel exits early, its log is printed.

## Config

Connections are stored in `~/.config/db/dbs.conf`, created with mode `600`. Set `DB_CONF` to use a different file.

Each line is pipe-delimited:

```
alias|host|port|user|default_db|password[|tunnel_cmd]
```

Lines starting with `#` and blank lines are ignored. `|` is not allowed inside any field.

> **Passwords are stored in plain text** in `dbs.conf`. Keep the file private and don't commit it.
