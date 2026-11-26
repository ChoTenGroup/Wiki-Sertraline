# Common Commands

Include the opening slash in game chat. Omit it in the server console. Angle brackets mark required arguments, and square brackets mark optional ones. Leave both kinds of brackets out when entering a command.

| Command | Use |
| --- | --- |
| `/sertraline help` | Show available subcommands |
| `/sertraline about` | Show the plugin version |
| `/sertraline reload` | Reload configuration. Check the console for errors naming individual files |
| `/sertralineitem browse` | Browse loaded items in game |
| `/sertraline manager help` | Show [item manager](../advanced/item-manager-system.md) commands |
| `/sertralineitem give <item-id> [player] [amount] [silent]` | Give items to an online player. Omit the player to give yourself one item |
| `/sertraline station sertraline:<station-id>` | Open a [crafting station](../advanced/crafting-stations.md) in game |
| `/sertralinegem list` | View sockets on the main-hand item. See [Gems](../advanced/gems.md) for socketing and removing gems |

For example, `/sertralineitem give beginner_sword Steve 2 true` gives the online player Steve two [Beginner's Swords](my-first-item.md) and suppresses the receipt message. Supply a player name when giving items from the console. The accepted amount is 1–2304.

The main, standalone item, and gem commands use `sertraline.command.main`, `sertraline.command.item`, and `sertraline.command.gem` respectively. Operators have access by default.
