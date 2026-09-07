# Development

## Checks and formatting

Use Python 3.10 or newer and Bash. Install the pinned tools in a virtual environment:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements-dev.txt
export PATH="$PWD/.venv/bin:$PATH"
python3 scripts/check.py
```

Run `python3 scripts/check.py --fix` to format Python and shell and normalize text whitespace. The same checks run on pushes and pull requests. XML is checked for well-formedness; game schemas, XPath matches and gameplay require separate X4 validation. Blender scripts are parsed and linted without importing Blender.

Use UTF-8, LF, a final newline, spaces and no trailing whitespace. Indent code with four spaces and workflow YAML with two. Use descriptive names, uppercase shell variables, constant-first equality comparisons and explicit boolean checks. Ruff's E712 rule is disabled to retain explicit boolean comparisons. Keep shell free of prose comments. Keep only short, non-obvious constraints in code; put explanations here. XML continuation attributes may align with their opening attribute. Preserve XPath selectors, savegame identifiers and embedded game expressions when applying formatting.

## Installation and publishing helpers

`install.sh` and `publish.sh` both source `lib/find_x4.sh`. The library searches usual Steam roots and additional library folders. `X4_PATH`, `X_TOOLS_PATH`
and `PROTON_PATH` override discovery. Proton Experimental is preferred when found; otherwise the helper uses the last matching Proton directory it encounters.

Installation replaces the extension directory with a copy of `extension/`. Refresh it after edits; X4 enumerates real extension directories, so a symlink does not substitute for installation. Restart the game after installing or removing.

Publishing stages a separate copy inside the game's extensions directory. The repository keeps its readable extension id; `steam/workshop-id` holds the numeric Workshop id. The helper changes only the staged manifest, runs the interactive WorkshopTool and restores the manual installation after success. On Linux it runs WorkshopTool through Proton and maps paths through drive Z. A failed upload can leave the staged copy behind; rerun `./install.sh` to restore it.

## Release metadata

`content.xml` uses an integer version multiplied by 100 and an ISO release date. The date matches the corresponding released entry in `CHANGELOG.md`. Development changes belong under `Unreleased`; they do not advance the manifest's release version or date. An unreleased scaffold may retain its initial creation date until its first release. Keep existing extension ids stable.

## Implementation constraints

### extension/libraries/charactergroups.xml

Two single-gender groups for the Teladi agent characters below.

Vanilla routes every Teladi-race agent through 'teladi.agent' -&gt; 'teladi.factionrepresentative', which holds the male and the female representative macro together. That is why a Teladi agent can already turn up either way - and also why the gender filter cannot force one: there is nothing to choose between, only the one group. Splitting the two macros apart gives it something to aim at.

### extension/libraries/characters.xml

Agent characters for the factions the gender filter could not otherwise reach.

Vanilla gives these factions a single agent character each, so the filter had nothing to choose between. What that meant differed per faction, and so does what is added here:

Quettanauts   'agent_kaori_arg' is bound to the male group, so a Quettanaut agent could only ever be male. The female group it needs ('kaori.crew.arg.female', three Argon female macros) already ships with Timelines - only the character entry was missing. Voice page 10602 is the one vanilla gives its female Argon agents. Teladi-race same story, through 'teladi.agent' -&gt; 'teladi.factionrepresentative'; the two groups it is split into live in charactergroups.xml next to this file.

Skills and voice pages are copied verbatim from each faction's vanilla agent, so a forced agent is the same character in every respect except which model it wears.

What keeps them out of the way is the tag on their category: 'drjele_agent' rather than 'agent'. Tags are free-form - "tag is created if it doesn't exist", per scriptproperties.xml - and nothing in the game looks for that one, so the &lt;select tags="tag.agent"&gt; vanilla spawns agents with never sees these entries. They exist only for the filter to name explicitly, and the mod at its default settings still changes nothing. (Leaving the category out entirely would do the same job on paper, but not one of the 412 characters the game ships is written that way, and an unindexed entry that cannot be resolved by id would leave a faction with no agent at all.)

The Paranid-race factions - Paranid, Holy Order, Alliance, Trinity, Buccaneers - and the Boron are missing on purpose, because no mod can give them a female agent. The game ships one set of Paranid bodies with no gender split, and character_paranid_rep_01_macro is flagged female="false". The Boron look like they have the split - boron.agent selects between boron.agent.male and boron.agent.female - but both of those groups resolve to the same and only Boron character macro, character_boron_suit_base_01_macro, which is also flagged female="false". The two group names are a vanilla convention, not two models.

The Quettanaut entry needs Timelines, for the group it references. Without it that faction does not exist either, so nothing ever asks for the character.

### extension/md/diplomacy.xml

Patches for the vanilla diplomacy script. Four independent inserts, each one reading its own setting and doing nothing at the setting's vanilla default.

Every setting is looked up the same way: the global the options menu writes wins, otherwise the table drjele_diplomacy_agents.xml publishes, otherwise the literal vanilla value - all of them global variables, so a configuration script that fails to load degrades to stock behaviour instead of breaking diplomacy. The table is tested for existence rather than the field inside it, so a setting edited to 0 in the file is still read as 0 and not mistaken for "not configured".

Experience multiplier. AgentExperience_RewardEvaluation is the only place in the game that hands out agent experience: it rolls $Experience from the risk of the action, then gives it to the skill the action prefers and a fifth of it to the other one. Scaling $Experience before that split therefore scales both skills, and the achievement check below it, in one place.

Experience for an agent that comes back injured. Vanilla pays experience only when the agent survives unhurt - a failed action still counts, an injury does not - and Risk_EvaluateOutcome is where the injury is applied, so it is where the consolation prize goes. The roll is the no-risk one (1-8, decreasing), scaled by the multiplier and by the percentage, then split evenly between both skills the way vanilla splits experience from diplomacy events.

Risk scale and the death switch, on the agent-action path. Success_Evaluation picks the injury and death chances from the risk of the action (low 15/0, medium 25/5, high 40/20, veryhigh 70/30) and then rolls against them; this lands between the two, after the last branch of the chain, so both numbers are already set and neither roll has happened yet.

The two settings stack in the order you would expect: the scale thins out both chances, then the switch can still zero the death one on its own, leaving the injury chance the scale left.

The same again on the diplomacy-event path. The game keeps a second copy of the whole risk block in DiplomaticEvent_Concluded, for the agent committed to an event option when the event goes the other way, and it is not shared with the one above - so a patch on one of them alone would leave half the ways an agent can get hurt untouched.

The stand-in the game sends on the mission. An agent on an operation is not the actor you hired:
AgentActor_CloneHandling builds a clone for the trip, copying the name and the five skills but picking the model itself, with the same &lt;select tags="tag.agent"&gt; and no reference to the agent it is standing in for. In vanilla that lands on the same character anyway, because the clone is seeded from the agent's own seed and draws from the same one or two entries. An agent this mod forced to a gender breaks that symmetry: at the Quettanauts, whose only tagged agent character is the male one, the clone came out male for a female agent - a different person wearing her name at the far end of the mission.

So the clone is picked the same way the agent was, except keyed on the agent's own gender rather than on the option: change the setting after hiring someone and her stand-in still looks like her. Factions with no pair in the table fall through to the vanilla select, untouched.

### extension/md/drjele_diplomacy_agents.xml

Configuration. Re-read on every savegame load, so editing a value below and reloading is enough - no new game required. Each one can also be overridden at runtime without touching this file, by setting the matching global variable:
&lt;set_value name="global.$DrJeleDiplomacyXpFactor" exact="5"/&gt; Those globals are also what the in-game options menu writes - see drjele_diplomacy_agents_options.xml.

Every default here is the vanilla value, so an installed-but-untouched mod changes nothing.

The settings are republished into one global table rather than read out of this cue directly. The code that reads them lives in patches inside the vanilla scripts, and a global variable is something those can look up defensively; a reference to a cue in this script would be a hard dependency, and a failed load here would then take diplomacy down with it. Every reader also carries the vanilla literal as a last fallback.

Faction id -&gt; [female character, male character] out of libraries/characters.xml. Keyed by faction.id rather than by the faction keyword on purpose: a keyword for a faction whose DLC is missing would not resolve, while an id string simply never matches.

The keys are written in the $name form because every string key in an MD table has to start with a '$' - a plain 'argon' is refused with "Failed to set table[].argon". The lookup side builds the same key with '$' + faction.id.

The first eleven are the factions whose agents ship in both genders. The rest come from the characters this mod adds. The Paranid-race factions - Paranid, Holy Order, Alliance, Trinity, Buccaneers - and the Boron are absent: the game holds no female model for either race, so their agents keep the vanilla pick whatever this is set to.

These four come from the characters this mod adds in libraries/characters.xml, for the factions vanilla gives a single agent character to. The Quettanaut male is the vanilla entry - only its female counterpart was missing.

### extension/md/drjele_diplomacy_agents_options.xml

In-game options, through SirNukes Mod Support APIs. Everything that touches that API lives in this file, so if the API is not installed nothing here ever runs and the mod keeps working off the constants in drjele_diplomacy_agents.xml.

All four callbacks only ever write the global override variables the patches already read, so the patched vanilla libraries know nothing about menus.

The API stores an option's value in the savegame under its $id, and hands it straight back to the widget as its start value. A stored value outside the slider's range fails widget validation and takes the whole Extension Options menu down with it, so the range and the unit of an option must never change under a $id that has already shipped - give it a new one instead. That is why these ids carry their unit.

Simple_Menu_Options only knows 'button' and 'slidercell', so the gender filter is a three-step slider rather than a dropdown; its mouseover carries the legend.

### extension/md/npc_agent.xml

Patch for the vanilla agent spawner, so the agents factions offer you can be filtered to one gender.

Vanilla picks the agent's character out of the CharacterDB with &lt;select tags="tag.agent"&gt;, which has no gender criterion - the DB holds a separate entry per gender instead. create_cue_actor's ref attribute takes a character id from an expression, so naming the entry ourselves is the whole trick; the faction id -&gt; [female, male] table lives in drjele_diplomacy_agents.xml.

Only the do_else branch is touched, the one that spawns an ordinary faction agent. The branch above it, which spawns a scripted agent from a definition handed in by the caller, is left alone, as is everything the spawned actor goes through afterwards.

Falls back to the vanilla pick whenever the filter is off, the faction has no gendered pair in the table (the Paranid-race factions and the Boron, which the game has no female model for), or the configuration script did not load - so no faction can ever end up without an agent.
