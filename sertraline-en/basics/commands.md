# Common Commands

Use the leading `/` in game chat and omit it in the server console. In this reference, `<...>` marks a required argument and `[...]` an optional one; do not type the brackets.

| Command | Use |
| --- | --- |
| `/sertraline help` | Show the main command tree available to you |
| `/sertraline about` | Show the installed plugin version and platform |
| `/sertraline reload` | Reload configuration and item definitions; check the console for file-specific errors |
| `/sertralineitem browse` | Browse loaded items in game |
| `/sertralineitem give <item-id> [player] [amount] [silent]` | Give an item to yourself or an online player |
| `/manager help` | Show [item manager](../advanced/item-manager-system.md) commands |
| `/sertraline station sertraline:<station-id>` | Open a [crafting station](../advanced/crafting-stations.md) in game |
| `/sertralinegem list` | List sockets on the item in your main hand; see [Gems](../advanced/gems.md) |

For example, `/sertralineitem give beginner_sword Steve 2 true` gives Steve two [Beginner's Swords](my-first-item.md) and suppresses the recipient message. The amount must be between 1 and 2304. When running the give command from the console, include an online player name; omitting the player gives the item to yourself only when run by a player.

{% hint style="info" %}
The item command also has aliases such as `/items`; the gem command has the `/sgem` alias. Operators have access by default. The main, item, manager, and gem command roots use `sertraline.command.main`, `sertraline.command.item`, `sertraline.command.manager`, and `sertraline.command.gem` respectively. `/sertraline help` only shows subcommands available to the sender.
{% endhint %}
