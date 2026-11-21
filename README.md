# darktide-mods-friends-list-fix
Friends List Fix is a small workaround for a vanilla regression from Darktide 1.13.0: the Friends tab of the Social menu can list the same player twice. It happens with players who are both your in-game (Fatshark) friend and your Steam or Xbox friend. The other Social menu tabs are not affected.

The mod picks one entry per player before the Friends tab is sorted and grouped, and keeps the in-game friend entry, so the player options (inspect, invite, remove friend and so on) keep working. It does not change the game's friends list itself, and there are no settings.

## What goes wrong

The game builds the friends list from two sources: your Fatshark friends, looked up by account id, and your platform friends, looked up by Steam or Xbox id. Each source keeps its own player entries, so one person can end up as two separate entries, one from each source.

Up to Darktide 1.12.5, merging the two lists dropped the second entry when it had the same account id or the same platform id as an earlier one, and kept the Fatshark entry. Since 1.13.0 the merge only drops an entry that is the very same object, so both entries survive. This is unchanged in 1.13.4. The Social menu also only removes identical entries that end up next to each other after sorting, which never catches two different objects.

When such a friend is playing Darktide, the game usually links the two entries by itself after a few seconds, and the duplicate goes away with the next refresh. The duplicates that stay are mostly friends who are online on Steam or Xbox but not in Darktide, and offline friends. This is worked out from the game's source code and tested against it in a harness.

## What the mod does

- Hooks `SocialMenuRosterView._prepare_list` and only touches the Friends tab, before vanilla copies, sorts and groups the list.
- Treats two entries as the same player when they have the same account id, or when one has no account id and uses the same Steam or Xbox account as an entry that has one. If two different accounts report the same platform account, nothing is merged.
- Keeps the entry that has its own account id (the Fatshark entry), as vanilla did up to 1.12.5. Many player options need the account id, and the platform entry often has none. Between two entries of the same kind it keeps the one in the better group (in game, then online, then offline), then the one with a platform id, then the first one.
- Leaves players hidden by cross-play settings hidden, exactly as vanilla does.
- Never changes the social service's friends list, so the other tabs, the main menu's friend count and the party join checks see vanilla data.
- Runs once per Social menu refresh (every 2 seconds) while the Friends tab is open, never per frame, and allocates no tables or strings per refresh.

The mod can only merge entries when the game knows that both belong to the same player, through the same account id or the same platform account. If it doesn't know that yet, both entries stay until it does.

## Compatibility

No special integration is needed. Mods that change how names look in the Social menu, such as True Level, keep working, because the mod only decides which entries are listed. Load order could only matter for another mod that also hooks `SocialMenuRosterView._prepare_list` and reads the Friends list before the original runs. No such mod is known.

## Testing

You need a player who is both your in-game friend and your Steam or Xbox friend, ideally one who is online on Steam or Xbox but not in Darktide. Open the Social menu: the Friends tab should list them once, in the right group, and inspect and invite should work. Disabling the mod in the mod options turns the hook off, which gives you a vanilla control run after the next refresh.

## Removal

This mod is meant to be temporary. Once Fatshark fixes the friends list merge, disable or uninstall it; it stores no settings or data.
