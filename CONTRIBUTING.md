# Contributing a pack

1. Fork this repo and add a folder `packs/<your-pack-id>/` with one file, `extra.json`. The id is 2–40 lowercase letters,
   digits or dashes, and must match the folder name.
2. Test it: drop your `extra.json` on your buddy (or *Extras → Install pack from file…*). Set
   `ASSISTAIL_EXTRAS_SPEED=60` before starting Assistail to make a "minute" one second while you try it.
3. Open a pull request. Only change your pack's folder: **don't** edit `index.json`, `index.json.sig` or `dist/`. A check
   runs the same validation as the app; the maintainer reviews the pack, then rebuilds and signs the catalog.

To update a pack, edit its `extra.json` **and raise its `version`** (otherwise installed copies won't update).

## The pack format (version 1)

```json
{
  "format": 1,
  "id": "stretch",
  "name": "Stretch reminder",
  "version": "1.0.0",
  "author": "Your name",
  "description": "Every so often he stretches and asks you to join in.",
  "minApp": "2.4.0",
  "icon": "🙆",
  "topics": ["stretch"],
  "settings": {
    "every": { "type": "minutes", "label": "Remind me every", "default": 60, "min": 15, "max": 240 }
  },
  "reminders": [
    {
      "every": "{every}",
      "activeTimeOnly": true,
      "quietWhen": ["fullscreen", "dnd", "focus", "chat"],
      "snooze": 10,
      "attention": 3,
      "do": [ { "trick": "stretch_yawn" }, { "say": ["Stretch with me, {name}?", "Up you get! 🦁"] } ],
      "onClick": [ { "say": "Nice! 💪" } ]
    }
  ]
}
```

- **settings**: `minutes`, `number` (with `unit`), `time` (HH:mm), `choice` (with `choices`), `bool`. Use them elsewhere as `"{key}"`.
- **reminders**: `every` (minutes) or `at` (a time, with `days`: `daily` or `weekdays`); `activeTimeOnly` (count only time
  the person is at the PC); `quietWhen`: `fullscreen`, `dnd`, `focus`, `chat`; `snooze` (minutes, for right-click);
  `attention` 1–5 (1 subtle, 2 normal, 3 he walks over, 4 his paw holds up a sign, 5 his paw boops the screen);
  `escalate` (if ignored, remind once more a little later, one level louder); `do` and `onClick` action lists.
- **routine** (one per pack, see `packs/pomodoro`): `steps` with `id`, `name`, `minutes`, `focus` (no quirks),
  `pose` (`sleep`), `confirm` (wait for a click before starting), `escalate`, `every` / `exceptEvery` (only on rounds that
  are / aren't a multiple of N), and `start`, `nudge`, `end` actions.
- **actions**: `say` (text, or a list to pick from at random; optional `seconds` and `attention`), `trick` (`shake_head`,
  `jump`, `stretch_yawn`, `tail_chase`, `wave`, `look_roar`, `walk_around`, `chase`, `butterfly`, `happy`, `look_up`,
  `swat`, `hover`, `sleep`), `count` (a counter for today).
- **placeholders in text**: `{name}` (the buddy's name), `{minutes}`, `{round}`, `{count:<counter>}`, `{<setting>}`.
  Built-in counter: `claude-tasks` (Claude Code tasks finished today).
- **topics** mute the buddy's own chit-chat on the same subject while the pack is on (`water`, `stretch`, `break`, `eyes`).

Limits: `extra.json` up to 64 KB, lines up to 200 characters, up to 10 actions per list and 10 routine steps. Anything
unknown makes the app show the pack as "needs a newer Assistail" rather than half-understand it.

## For the maintainer: rebuilding the catalog

From the Assistail repo, with the maintainer's signing key (kept off every repo):

```
dotnet run --project tools/ExtrasCatalog -c Release -- build <path-to-assistail-extras>
```

This validates every pack, writes `dist/*.zip` and `index.json`, and signs it. Commit those files and push.
