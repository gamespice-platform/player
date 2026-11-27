[![Player - GameSpice addon](docs/banner.png)](https://moddingcommunity.com/a/37-gs-player)

One shared record per player, so team, class, alive state and stats stop living in four separate dictionaries that disagree after a reconnect.

`GSPlayerRoster` holds the rows. `GSPlayer` is the node that represents a player in the scene. `GSPlayerComponent` is the base class other player addons extend.

## Install

1. Copy `addons/gs_player/` and `addons/gs_core/` into your project.
2. Enable both in *Project > Project Settings > Plugins*.

Needs Godot 4.7 or newer.

## Use

```gdscript
var roster := GSPlayerRoster.new()
roster.config = GSPlayerConfig.new()
add_child(roster)

roster.join("ada", "Ada", peer_id, tick)
roster.set_team("ada", &"blue")
roster.set_alive("ada", true)

# Once a tick or once a second: frees seats whose reconnect window is up.
roster.advance(tick)
```

On a client, use the same class with `authoritative = false`:

```gdscript
mirror.apply_wire(payload)   # the whole roster, on connect
mirror.apply_row(row)        # one changed row afterwards
```

A mirror decides nothing. Every change method on it fails with `CODE_FORBIDDEN`.

## Reconnecting

A dropped connection keeps the player's seat. Calling `join` with the same key inside the reconnect window is a rejoin: same record, team, score and join tick, new peer id. Set `reconnect_window_sec = 0` to remove players immediately, which suits a lobby. Held seats count towards `max_players` by default, so a stranger cannot take the seat of a player who is reconnecting.

## Settings

All in `GSPlayerConfig`, layered like every GameSpice Core config (defaults, JSON file, environment, command line).

| Setting | Default | Meaning |
| --- | --- | --- |
| `max_players` | 0 | Player limit. 0 means unlimited. |
| `reconnect_window_sec` | 60 | How long a dropped player's seat is held. 0 removes immediately. |

## Nodes

`GSPlayer` extends `Node`, not a body, so it works in 2D and 3D and for players with no body (spectators, the dead, passengers). It finds its body and components in its children.

`GSPlayerComponent` finds its player by walking up the tree, so no `NodePath` is needed. Subclasses override `_on_bound` and `_on_unbound`, not `_ready`, because the key is assigned after `_ready`. `is_active()` checks the bind and the `local_only` rule.

| Class | Purpose |
| --- | --- |
| `GSPlayerRecord` | One player row: key, name, peer, team, class, character, alive, counters, metadata. |
| `GSPlayerRoster` | Joins, rejoins, held seats, expiry, stable key order. Authoritative on the server, mirrored on clients. |
| `GSPlayerConfig` | Capacity, reconnect window, name handling. |
| `GSPlayer` | The scene node. |
| `GSPlayerComponent` | Base class for player components. |

See [docs/design.md](docs/design.md) for details.

## Used by

All fourteen games vendor it:

- [Fatal Contact](https://zeesgames.com/fatal-contact)
- [Airdrift](https://zeesgames.com/airdrift)
- [Playlot](https://zeesgames.com/playlot)
- [Hungario](https://zeesgames.com/hungario)
- [Buses From Hell](https://zeesgames.com/buses-from-hell)
- [Smash Copter](https://zeesgames.com/smash-copter)
- [Zombie Zone](https://zeesgames.com/zombie-zone)
- [Hide n Seek](https://zeesgames.com/hide-n-seek)
- [Prop Hunt](https://zeesgames.com/prop-hunt)
- [Murder Mystery](https://zeesgames.com/murder-mystery)
- [Deathrun](https://zeesgames.com/deathrun)
- [Wipeout](https://zeesgames.com/wipeout)
- [Dangerous Delivery](https://zeesgames.com/dangerous-delivery)
- [Look At Me](https://zeesgames.com/look-at-me)

The whole collection is at [zeesgames.com](https://zeesgames.com).

## Status and licence

Part of the [**GameSpice**](https://gamespice.net) platform: a family of Godot addons, MIT licensed and built with Claude Code. Its headless suite is `res://examples/player_selftest.tscn`. The roster is not a shelf item: the games in [Zee's Games](https://zeesgames.com) are built on GameSpice addons and run on live servers, so reconnects and seats have met real players. Report problems on the [issue tracker](https://github.com/gamespice-platform/player/issues).

MIT licence, see [LICENSE](LICENSE). The only closed part of the [**GameSpice**](https://gamespice.net) platform is the web back end; you can use your own instead of GameSpice's.
