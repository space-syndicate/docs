# Server Hosting Tutorial

This is a tutorial/brain dump of how to set up an SS14 server. This guide will cover everything you need, from private servers to play with your friends, to production servers for proper server hosts.

There are two methods you can use. A "bare" server that just runs the game server directly, or `SS14.Watchdog` which handles updates and runs the game server for you. We recommend the latter for more proper deployments and also if you want to be listed on the hub in the launcher.

If you have any questions, ask on Discord and/or ping PJB if you really don't get an answer.

## Useful Links
All of the important links on this page in one convenient place.
* [Config Reference](../tips/config-file-reference.md)
* [.NET 7 Runtime](https://dotnet.microsoft.com/download) (Also included in full .NET 7 SDK)
* [ASP.NET Core 7 Runtime](https://dotnet.microsoft.com/en-us/download/dotnet/7.0) (Also included in full .NET 7 SDK)
* [SS14.Watchdog](https://github.com/space-wizards/SS14.Watchdog/)
* [Official Builds](https://central.spacestation14.io/builds/wizards/builds.html)
* [Wizard's Den Infrastructure Reference](../../community/infrastructure-reference/wizards-den-infrastructure.md) (server specs)
* [Public Hub Server Rules](../../community/space-wizards-hub-rules.md)

## Install Dependencies

Regardless of the hosting method you choose, the server is not self-contained (and neither are client builds) and therefore needs a [.NET 7 Runtime](https://dotnet.microsoft.com/download) installed. You only need "x64" under "run console apps" not "hosting bundle" from the downloads page.

For other services such as `SS14.Watchdog` you ALSO need the [ASP .NET Core 7 Runtime](https://dotnet.microsoft.com/en-us/download/dotnet/7.0) (included in .NET 7 SDK).

## Set Up Server

1. Download the latest version of the server from [our builds page](https://central.spacestation14.io/builds/wizards/builds.html), for your operating system. If you are running custom code, or a build is not available for your platform, see [Custom Code](#custom-code) below.
2. Extract that to a directory somewhere.
3. Run `Robust.Server.exe` (or `Robust.Server` [via terminal on macOS/Linux](#running-the-server-on-macos-or-linux))
4. [Forward your network ports](#port-forwarding)
5. Give your friends and yourself your IP address and have them paste it into the "direct connect" dialog in the launcher.

This will not, of course, handle automatic restarts (in case of a crash) or updates like the watchdog would. This also won't list your server publicly on the hub as advertising defaults to off. If you wish to have your server listed on the hub please read `Bare Server Configuration` below.

```admonish info
Note that when you run a server like this, the build information that the launcher needs (so it has a version of the client to download) points to our central server. **We do not keep those builds around forever** so eventually people will no longer be able to connect to the server. Obviously this is not a problem if you want to set up a quick and dirty private server for your friends. Just make sure to download a new version of the server at least every week.
```

### Port Forwarding

The server needs network ports to be forwarded so that people can connect. By default, the game server uses two ports:
* UDP `1212` is used for main game netcode. This is necessary for the *client* to be able to connect to the server. This can be configured with the `net.port` configuration variable.
* TCP `1212` is a HTTP status API. This is also necessary for the *launcher* to be able to connect to the server. You do not need this to connect with a bare client. This can be configured with the `status.bind` configuration variable (which takes in a string like `*:1212` or `127.0.0.1:3000`).

For more information about how to forward your ports, see: [Port Forwarding](../../server-hosting/port-forwarding.md)

#### Advanced Port Forwarding

You can slap the HTTP status API behind a reverse proxy if you want. This is recommended for production servers since then you can do HTTPS (slap it behind nginx and turn on HTTPS). Note that if you do this you have to set the `status.connectaddress` config variable to specify the UDP address the main netcode should connect to. This has to look like this: `udp://server.spacestation14.io:1212` (for our server, obviously substitute with your params). 

### Custom Code
You need to [set up a development environment](./setting-up-a-development-environment.md) in order to produce a server build for custom code. After you do that, you need to generate the server build by running:

`python Tools/package_server_build.py --hybrid-acz`

Check the `release/` folder for a packaged server for your custom codebase.

## Configure Your Server

You can configure settings in the server by editing the config file, `server_config.toml`. The config file is TOML which is basically INI ~~except better specified, somewhat more powerful, easier to misuse, and more annoyingly opinionated (comments NEED their own line)~~.

Settings have one key they fall under and then the name. So if I say `game.lobbyenabled` it goes under the `[game]` header like so:
```toml
[game]
lobbyenabled = true
```

Got that? Good.

**Some sane defaults you might want to set for your server if you actually intend to host this properly:**

```admonish warning
Please read through the comments here so you have a solid grasp of what you're doing.
```

```toml
[net]
# Reducing tickrate to 30 is basically unnoticeable
# but reduces server, client and network load dramatically.
tickrate = 30

[game]
# Changes the name of the server as it appears in the lobby and launcher.
hostname = "Foo Station"
# Enables the lobby instead of straight up throwing clients into a game.
lobbyenabled = true

[auth]
# Enforces authentication so that ALL connecting clients must have a proper account.
# Otherwise, guest logins are allowed.
# Possible values here are: 0 (optional), 1 (required), 2 (disabled)
mode = 1
```

See [Config File Reference](../tips/config-file-reference.md) for a somewhat more thorough guide on server configuration.

### Setting Rules
By default, the server ships with the rules that are used on Wizard's Den servers. To set custom rules for your own server:

1. Add a file with your rules to the `Resources/ServerInfo/` directory.
2. Set the `server.rules_file` CCVar with the base name of your rules file (just the file name without the leading path).

### Public Hub Server - Getting your server on the launcher's list

1. Read  the [hub server rules](../../community/space-wizards-hub-rules.md) before putting your server on the hub. Advertising to the hub constitutes acceptance of the hub rules.

2. Pick tags for your server based on the [standard tags](../../robust-toolbox/server-http-api.md#standard-tags).

3. Add the following lines to your [server configuration](../tips/config-file-reference.md):

    ```toml
    [hub]
    advertise = true
    # Uncomment to change the server URL advertised on the master server list.
    # Use this if you want an ss14s:// URL or have configured the server behind a reverse proxy or any of that.
    # Defaults to "ss14://[public server IP]:networking port"
    # server_url = "ss14://..."
    tags = "" # comma separated list of tags
    ```

### Bare Server Build Configuration

If you *do* want to set up a more permanent server, you will have to re-host the client downloads somewhere. Anywhere accessible via a plain URL is fine.

You will want to edit the server config file (`server_config.toml`) to add the following to it:

```toml
[build]
# Download locations of the entire client build in a zip, as a HTTP (or HTTPS) URL.
download_url = ""
```

### Admin Privileges

By default, no admin privileges are set. A privileged administrator can give out permissions to other admins with the `permissions` console command in-game, but that has a chicken-and-egg problem. To get initial `+HOST` administrator permissions to your server, you can use one of the following three methods:

```admonish danger
`+HOST` privileges are **extremely dangerous** to give and should only be given to people who already have access to your computer or server.

**Giving somebody `+HOST` allows them to completely take over your server and/or computer.** 
```

* If you connect to the game server over localhost (IP `127.0.0.1` or `::1`), the game will automatically give you full host privileges. This can be disabled with the `console.loginlocal` CVar.
* If you set the `console.login_host_user` CVar to your user name, you will be given host when you connect.
* You can use `promotehost` command from the server console (e.g. `promotehost PJB`) to temporarily give a connected client host.

## Performance Tweaks

Here are some settings you probably want to enable on your server to improve performance:

Environment variable to enable full dynamic PGO, which drastically improves performance at the cost of marginally higher startup time:
```
DOTNET_TieredPGO: 1
DOTNET_TC_QuickJitForLoops: 1
DOTNET_ReadyToRun: 0
```

Environment variable to enable AVX operations across the codebase. Depending on your processor, this might hurt performance instead of improving it, otherwise it may improve atmos performance.
```
ROBUST_NUMERICS_AVX: true
```

You can set environment variables from the watchdog, see below.

## Watchdog

This is for people running their own codebase and server and/or those who want a more robust hosting solution.

[`SS14.Watchdog`](https://github.com/space-wizards/SS14.Watchdog/) (codename Ian) is our server-hosting wrapper thing, similar to TGS for BYOND (but much simpler for the time being). It handles auto updates, monitoring, automatic restarts, and administration. We recommend you use this for proper deployments.

### Installation
1. Download or clone the latest `SS14.Watchdog` source code (linked above)
2. Run `dotnet publish -c Release -r linux-x64 --no-self-contained`, replacing `linux-x64` with an identifier for your platform
3. Copy the `SS14.Watchdog/bin/Release/net6.0/linux-x64/publish` directory (check this path for your platform as necessary) to where you want to store your SS14 server files.

### Configuration
Configure the watchdog to add servers. Edit `appsettings.yml` inside the installed watchdog directory.

1. Uncomment and set `BaseURL`. This is the address that your game servers will use to communicate with the watchdog. The default value is fine for small setups. For example:
    ```yml
    BaseUrl: https://builds.spacestation14.io/watchdog/
    ```

2. At the bottom of the config file, add a `Servers` YAML block, and then a `Instances` block inside of that. Inside the `Instances` block, add a block for each server that you plan to serve from this watchdog. For example:

    ```yml

    Servers:
      Instances:
        wizards_den: # ID of your server.
          Name: "Wizard's Den" # Name of the server - Note that this is NOT the name of the server on the hub, that is set for each server under game.hostname in their respective config.toml files.
          ApiToken: "foobar" # A secret password that you make up (API token) for the watchdog to control this server (e.g. update, restart)
          ApiPort: 1212 # API port OF THE GAME SERVER. This has to match the 1212 HTTP status API (described below). Otherwise the watchdog can't contact the game server for stuff.

          # Auto update configuration. This can be left out if you do not need auto updates. Example is for our officially hosted builds.
          # See below for alternatives.
          UpdateType: "Manifest"
          Updates:
            ManifestUrl: "https://central.spacestation14.io/builds/wizards/manifest.json"

          # Any environment variables you may want to specify.
          EnvironmentVariables:
            Foo: bar
        wizards_den_two:
          # Name of the second server
          Name: "Wizard's Den 2"
          # etc...
    ```

### Server Instance Folder

The watchdog will automatically create a folder structure for each server instance. This is at `instances/<instanceId>`, e.g. instances/wizards_den / instances/wizards_den_two, relative to the current working directory when executing the watchdog.

Each instance folder has the following files and folders:

* `binaries/`: Is used to store client binaries when using the "Local" update type, see below.
* `bin/`: Contains the actual extracted server binaries.
* `data/`: Stores server data like player preferences.
* `config.toml`: Is the config file the server will load (the watchdog overrides the default location, `server_config.toml` next to the .exe, to avoid it getting deleted when the server resets).

```admonish info
Note that although the watchdog handles server updates you may still want to setup the config.toml as per the `Bare Server Configuration` section above.
```

### Update types
The watchdog supports 4 different types of update types:
1. Jenkins
2. Local
3. Git
4. Manifest

If you wish to obtain more information you can browse the SS14.Watchdog repo to see how they work.

TODO: Explain the update providers better and explain hybrid ACZ.

#### 1. Jenkins Updates
TBC

#### 2. "Manual" Local Updates

You can use the watchdog with local files, and have it automatically generate the necessary build information. This will also host the client binaries for you.

To configure this, use the following update configuration in your `appsettings.yml`, in the entry for your server instance:

```yml
UpdateType: "Local"
Updates:
  Version: "foobar" # Version string to use.
```

The watchdog will automatically host client binaries. Where does it pull them from? The `binaries/` folder in your server instance folder! Note that for this to work, the watchdog HAS to be publically accessible via `BaseUrl` in the config file.

You can edit the `Version:` specified in the config to tell the launcher that it should update the game next time you connect.

You will have to manually move files around and extract the server binaries.

#### 3. Git
This provider has 3 options available
- BaseUrl: This is the url for the repo, i.e. https://github.com/space-wizards/space-station-14
- Branch: The branch to checkout from the repo. This defaults to master
- HybridACZ: `true` or `false`. You most likely want to keep this `true`

#### 4. Manifest
TBC

#### Custom Auto Updates

Not supported, but it should be relatively trivial to edit the code for `SS14.Watchdog` to add support for whatever update mechanism you need. See `UpdateProvider.cs`.

### Server Build Configuration

The launcher needs to download the client binary to be able to run the game. It gets information about this client binary from the game server via an info API.

The information returned from this API is configured in two ways: `build.json` and `build.*` configuration variables.

`build.json` is a file that gets put next to the server executable automatically by the build system. This is how the server knows what the build info is when just downloading a bare server zip. (note that this is NOT done by `package_release_build.py`, since it relies on extra build information. `gen_build_info.py` does it in a separate step)

The second option is by specifying configuration variables (from command line or config file, both work):

```toml
[build]
# "Identifier" of your codebase. This is used by the launcher to manage installations.
# Try to keep this unique between different codebases.
# Nothing will break if you don't (or if there's a malicious actor),
# but the launcher WILL be forced to redownload files more often than otherwise necessary.
fork_id = ""

# Version string of the current build running on the server.
# This just prompts the launcher to redownload if it's different.
version = ""

# Version of the engine to download.
# Engine versions are hosted by us and will probably stay up forever.
# At least, as long as they are not found to be vulnerable to any security exploits, then we may pull them.
engine_version = ""

# Download locations of the entire client build in a zip, as a HTTP/HTTPS URL.
download_url = ""

# SHA256 hash of client zip files specified above.
hash = ""
```

Note that `SS14.Watchdog` specifies *most* of it for you if you have configured it with auto updates (depending on update provider). It notably cannot provide `engine_version` or `fork_id` version, so you're best off specifying the former in build.json (your build system should be non garbage for this) and the latter in a config file.

## Optional Infrastructure
Things that aren't necessary for small/private servers, but strongly recommended for forks or larger production servers.

### PostgreSQL Setup

SS14 uses an SQL database to store server data like player slots. By default, an **SQLite** database is automatically used which is sufficient for local testing and small servers. If you want the ability to share the database between multiple servers or such however, the server also supports connecting to **PostgreSQL**.
Support for MySQL/MariaDB isn't currently planned, but we will accept contributions.

Relevant configuration properties, along with default values:

```toml
# Server config file
[database]
# Database type to use. Can be "sqlite" or "postgres" currently.
engine = "sqlite"

# Path to store the database at when using SQLite. Note that is NOT a disk path.
# This is relative to the server data directory, which is specified by --data-dir when starting the server from the command (or automatically set by SS14.Watchdog)
sqlite_dbpath = "preferences.db"

# PostgreSQL database configuration, should be self explanatory.
pg_host = "localhost"
pg_port = 5432
pg_database = "ss14"
pg_username = ""
pg_password = ""
```

The game server automatically does migrations when it starts up, you do not have to do them manually.

### Prometheus Metrics

SS14 supports hosting a metrics server that [Prometheus](https://prometheus.io/) can scrape, with which you can then make fancy graphs in [Grafana](https://grafana.com/) or such. You can find our Grafana dashboards [here](../../community/infrastructure-reference/grafana-dashboards.md), in case they happen to be useful.

To configure this, you can use the following config variables:

```toml
[metrics]
enabled = true
# Address to bind the metrics server to, use "*" for all local interfaces
host = "localhost"
# Port to bind the metrics server to
port = 44880
```

You can then scrape this with the following Prometheus config (for example):

```yml
global:
  scrape_interval: 1s
  evaluation_interval: 1s

scrape_configs:
  - job_name: "wizards_den_us_west"
    static_configs:
      - targets: ["localhost:44880"]
```

### Loki Logging

SS14 also supports pushing structured log data to [Loki](https://grafana.com/oss/loki/). Because this is modern DevOps crap the website doesn't say what it actually does but when combined with Grafana you can go and look at and filter logs in a debatably more sane way than bare text files.

No, you do not need Promtail set up for this to work. SS14 pushes directly to Loki.

To configure this, you can use the following config variables:

```toml
[loki]
enabled = true
# HTTP address of the Loki server.
address = "http://localhost:3100"
# Name of this server, gets included in all log messages.
name = "wizards_den_us_west"
# Parameters for HTTP Basic authentication, if you have Loki configured behind it.
# If left out, no authentication will be attempted.
# username = ""
# password = ""
```

## Troubleshooting

### Unable to advertise to hub / people cannot connect

People aren't able to connect to your server OR you get the following error in your server console:

```
[ERRO] hub: Error status while advertising server: [UnprocessableEntity] "Unable to contact status address"
```

This means your server is not accessible from the outside internet. Make sure you have followed the guide to [Port Forwarding](../../server-hosting/port-forwarding.md).

### SS14.Watchdog

#### Server keeps restarting every 30 seconds

This means the server isn't communicating with the watchdog correctly and the watchdog is forced to assume that the server is locked up or similar. This happens if `BaseUrl` in the watchdog configuration is set incorrectly or otherwise inaccessible by the game server.

#### `System.IO.FileNotFoundException: Could not load file or assembly 'Mono.Posix.NETStandard, Version=1.0.0.0, Culture=neutral` (...)

Current working theory is that this is caused by improper `dotnet publish` settings.
The below set of test results should help explain.

```
dotnet publish -c Release -r linux-x64 --no-self-contained SS14.Watchdog -o test
 RESULT: Mono.Posix.NETStandard.dll included, System.dll not included (as expected)

dotnet publish -c Release -r linux-x64 SS14.Watchdog -o test
 RESULT: Mono.Posix.NETStandard.dll included, System.dll included

dotnet publish -c Release SS14.Watchdog -o test
 RESULT: Mono.Posix.NETStandard.dll not included, System.dll not included
```

Since Watchdog uses `Mono.Posix.NETStandard.dll` to mark executables as executable on Linux and Mac OS X, it's important to have it around on those OSes.

### Running the server on MacOS or Linux

Open a terminal in the unzipped build directory (it should have the Robust.Server file in it.)
Type `./Robust.Server` then hit enter. If you see a bunch of stuff being printed to the screen and it doesn't say error, then the server is running.

### Additional Troubleshooting

[Troubleshooting](../tips/troubleshooting-faq.md#server)
