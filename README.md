# Diplomacy Agents for X4: Foundations

<p align="center">
  <img src="extension/preview.jpg" alt="Diplomacy Agents" width="512">
</p>

X4 9.00 gives every faction an agent you can hire and send on negotiation and espionage actions. Those agents learn slowly — an action pays 1 to 15 experience and the top rank, Spymaster, sits at 400 — and the interesting actions are the ones that can get the agent injured for twenty minutes or killed outright, taking their experience with them.

This mod turns those numbers into sliders: how fast agents learn, whether an injured agent gets anything out of the attempt, how dangerous a failed action really is, and which gender of agent the factions put up for hire.

**Every slider starts at the vanilla value**, so installing the mod changes nothing on its own — you turn up only the parts you want. It patches four spots in two vanilla scripts and adds no cues of its own beyond a configuration block, so it can go on and come off an existing savegame.

**Requires X4: Foundations 9.00.** No DLC required — the DLCs are supported where they add agents, but nothing depends on them.

## Install

```bash
./install.sh
```

The helper copies `extension/` into the game's `extensions/<extension-id>`
directory, using the id in `extension/content.xml`. It searches the usual Steam layouts and additional library folders. To choose an installation:

```bash
X4_PATH="/path/to/X4 Foundations" ./install.sh
```

Restart X4 after installing or updating. To remove the manual installation:

```bash
./install.sh --uninstall
```

## What it changes

| Setting                     | Default | Range                   | What it does                                                                                                                                                          |
|-----------------------------|---------|-------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Agent experience multiplier | `1`     | 1–20                    | Multiplies the experience an agent earns per action. Applies before the game splits it between the two skills, so the fifth that goes to the off-skill scales with it |
| Experience when injured     | `0`     | 0–100 %                 | An agent that comes back injured gets this percentage of a no-risk experience roll, split between both skills. Vanilla gives nothing                                  |
| Agent risk                  | `100`   | 0–100 %                 | Scales the injury and death chances a failed risky action rolls against. At `0` an agent can never be hurt or killed                                                  |
| Agents can die              | `on`    | on / off                | Off caps the worst a risky action can do at an injury. It does not change how often one happens — that is the slider above                                            |
| Agent gender                | `0`     | 0 any, 1 female, 2 male | Which agents factions offer you from now on                                                                                                                           |

A detail worth knowing before you touch the second slider: **vanilla already pays experience for a failed action**, as long as the agent walks away from it. What it does not pay for is an action that puts the agent in the infirmary — that is the gap this fills.

The gender filter reaches fifteen of the twenty-one factions that field an agent. Eleven of them ship an agent character in each gender and need nothing from the mod: Argon, Antigone, Hatikvah, Vigor Syndicate, Riptide Rakers, Zyarth Patriarchy, Free Families, Court of Curbs, Segaris Pioneers, Terran Protectorate and Yaki.

Four more get their gendered characters from the mod, because vanilla gives them a single agent character each and the filter had nothing to pick between. The Teladi-race factions — Teladi Company, Ministry of Finance, Scale Plate Pact — draw theirs from a group holding both genders, so their agents vary on their own but could not be forced; the mod splits that group in two. The **Quettanauts** were worse off: their one character is bound to the male group, so a Quettanaut agent could only ever be male. Timelines already ships the female group it needed — only the character entry was missing, and that is what the mod adds.

**Six factions cannot be done at all**, and no mod can do better: Paranid, Holy Order, Alliance, Trinity, Duke's Buccaneers and the Boron. Their agents come from a race the game holds no female model for. `character_paranid_rep_01_macro` is flagged `female="false"`, and
`assets/characters/paranid/bodies` holds one set of bodies with no gender split where Argon has
`char_arg_f_` and `char_arg_m_` side by side. The Boron look like an exception — `boron.agent`
selects between `boron.agent.male` and `boron.agent.female` — but both groups resolve to the same and only Boron character macro, `character_boron_suit_base_01_macro`, also flagged `female="false"`. Those two group names are a vanilla convention, not two models.

What keeps the added characters out of the way is the tag on their category: `drjele_agent` instead of `agent`. Tags in X4 are free-form — *"tag is created if it doesn't exist"*, per
`scriptproperties.xml` — and nothing in the game looks for that one, so the
`<select tags="tag.agent">` the game spawns agents with never sees them. They exist only for the filter to name explicitly, and the mod at its defaults still changes nothing. It also only applies to agents spawned from that point on: agents you already know keep the gender they were born with. Fire one and the faction sends a new one along 30 to 60 minutes of game time later — that delay lives in `AgentCleanUp`, and is a random pick inside the range.

## Configuration

### In game

If [SirNukes Mod Support APIs](https://steamcommunity.com/sharedfiles/filedetails/?id=2042901274)
is installed, **Extension Options → Diplomacy Agents** gets the four sliders above. Changes apply to the next action or the next agent — no reload.

That API only knows buttons and sliders, which is why the gender filter is a three-step slider rather than a dropdown.

The dependency is optional and declared as such. Without it the mod behaves exactly the same, just configured by the file below instead. Everything that touches the API lives in its own script,
[`drjele_diplomacy_agents_options.xml`](extension/md/drjele_diplomacy_agents_options.xml), which writes the same global variables the patches already honour.

Settings are stored in the savegame, so each save carries its own.

### In the file

The values live at the top of
[`extension/md/drjele_diplomacy_agents.xml`](extension/md/drjele_diplomacy_agents.xml)
(`$XpFactor`, `$InjuredXpPercent`, `$RiskPercent`, `$AgentsCanDie`, `$AgentGender`). They are re-read on every savegame load, so editing one and reloading is enough — no new game.

Each can also be overridden at runtime, without editing the file, from a cheat menu or any other mod. These are the same variables the options menu writes:

```xml

<set_value name="global.$DrJeleDiplomacyXpFactor" exact="5"/>
<set_value name="global.$DrJeleDiplomacyInjuredXpPercent" exact="50"/>
<set_value name="global.$DrJeleDiplomacyRiskPercent" exact="25"/>
<set_value name="global.$DrJeleDiplomacyAgentsCanDie" exact="0"/>
<set_value name="global.$DrJeleDiplomacyAgentGender" exact="1"/>
```

## How it works

Four inserts, each reading its own setting and doing nothing at that setting's default.

| Patch                       | Vanilla hook                                                                                                 | Why there                                                                                                                                                                                                                                                                                                                        |
|-----------------------------|--------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Experience multiplier       | `AgentExperience_RewardEvaluation` in `md/diplomacy.xml`                                                     | The only place in the game that hands out agent experience. Scaling the roll before the game splits it covers both skills and all 22 call sites at once                                                                                                                                                                          |
| Experience when injured     | the `injured` branch of `Risk_EvaluateOutcome`, same file                                                    | Vanilla pays experience only under `agentresult.survived`; this is where the injury itself is applied                                                                                                                                                                                                                            |
| Risk scale and death switch | `Success_Evaluation` **and** `DiplomaticEvent_Concluded`, same file, after the last branch of the risk chain | Both chances are set by then and neither die has been rolled yet. The game keeps two separate copies of that risk block — one for agent actions, one for the agent committed to a diplomacy event — so patching one alone would leave half the ways an agent can get hurt untouched                                              |
| Gender filter               | the ordinary-agent branch of cue `AddAgent` in `md/npc_agent.xml`                                            | Vanilla picks the character with `<select tags="tag.agent">`, which has no gender criterion — the character database holds one entry per gender instead, and `create_cue_actor`'s `ref` takes a character id from an expression                                                                                                  |
| Mission stand-in            | `AgentActor_CloneHandling` in `md/diplomacy.xml`                                                             | An agent on an operation is a clone the game builds for the trip. It copies the name and skills but picks its own model, so a forced agent would meet you at the far end as a different person. Keyed on the agent's own gender rather than on the setting, so changing the setting later does not restyle someone already hired |
| Missing characters          | `libraries/characters.xml` and `libraries/charactergroups.xml`                                               | Seven agent characters and two single-gender Teladi groups, for the four factions vanilla gives one agent character to                                                                                                                                                                                                           |

The faction id → `[female, male]` table is keyed by `faction.id` rather than by faction keywords on purpose: a keyword for a faction whose DLC is not installed would not resolve, while an id string simply never matches. That is also why the mod needs no DLC dependencies and does not care about extension load order.

Every setting is looked up the same way — the global the options menu writes wins, then the table the configuration script publishes, then a literal vanilla fallback. All three are global variables rather than references into this mod's own cues, so a configuration script that fails to load degrades to stock behaviour instead of taking diplomacy down with it.

## Debugging

Add `-debug scripts -logfile debuglog.txt` to the game's launch options. A diff that silently does nothing looks exactly like a mod that is not working, and a patch whose XPath finds no node says so outright:

```
[=ERROR=] 0.00 No matching node for path '...' in patch file 'extensions\drjele_diplomacy_agents\md\diplomacy'. Skipping node.
```

so `grep -i drjele_diplomacy_agents debuglog.txt` after a start is the whole check — silence there means all four patches landed.

The log is written to the X4 user directory, next to your savegames —
`Documents/Egosoft/X4/<id>/` on Windows, `~/.config/EgoSoft/X4/<id>/` on Linux (or under
`~/snap/steam/common/.config/EgoSoft/X4/<id>/` for the Steam snap).

## Status

Patched-file structure and all four XPath selectors verified against the 9.00 game files; in-game verification pending.

## Publishing to the Steam Workshop

Install **X Tools** (Steam app 282160) and keep Steam running and logged in with an account that owns X4. On Linux, install Proton as well; on Windows, run the helper from Git Bash, MSYS or Cygwin.

```bash
./publish.sh publish
./publish.sh update "what changed"
```

Use `publish` once, then `update` with a change note. `X4_PATH`,
`X_TOOLS_PATH` and `PROTON_PATH` override automatic discovery. The staging location must contain an `extensions` directory.

The first upload records the numeric id in `steam/workshop-id`; retain that file for future updates. The readable id in the repository's `content.xml`
stays unchanged. After publishing, open the printed Workshop URL, complete any required Steam agreement and choose the item's visibility. Avoid keeping both the manual installation and a subscription to the same mod enabled.

Update the manifest version and release date together with `CHANGELOG.md`
when releasing. See [Development](DEVELOPMENT.md) for staging, platform and release conventions.

## Development

See [DEVELOPMENT.md](DEVELOPMENT.md) for setup, code style, validation and release conventions.

## Legal

MIT, see [LICENSE](LICENSE). Non-commercial fan project; X4: Foundations belongs to Egosoft GmbH and this project is not affiliated with or endorsed by Egosoft.
