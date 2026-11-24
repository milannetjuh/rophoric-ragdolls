# Rophoric Ragdolls

**Euphoria-style active ragdoll physics for Roblox.**

Rophoric Ragdolls is a character physics engine for Roblox, written in Luau. Characters react to being hit the way real people do: the head snaps back, the body takes stumbling steps to keep its balance, a hand goes to the face, and the body recovers or falls. All of it comes from physics, not from canned animations.

> **Status: early development.** The engine is being built. This README describes the goal; features are added as they're finished.

---

## The idea

Most Roblox ragdolls are either fully animated (stiff, the same every time) or fully limp (floppy, lifeless). Rophoric sits in between, like the active ragdolls in **GTA IV**, **Red Dead Redemption 2**, **Half Sword** and **Blade & Sorcery**:

- **The body is always physical.** Every hit, push and fall is simulated, so no two reactions are the same.
- **Physical, but trying to act like a person.** Simulated muscles steer the body to keep its balance, protect itself and move like a person, so the motion looks human, not like a doll.
- **Smooth.** Motion flows from one moment to the next, with no snapping, popping or teleporting between states.

## Planned features

**Core engine**
- [ ] Active ragdoll R15 characters that stand, walk and turn while staying fully physical
- [ ] Hit reactions that depend on where the hit lands, how hard, and from which side
- [ ] Balance and stumble steps when pushed
- [ ] Protective reactions: a hand to the face, hunching over, backing away
- [ ] Knockdowns, lying on the ground and getting back up
- [ ] Damage, knockouts and death
- [ ] Gore: blood, bruises and wounds, dismemberment and slicing, with bodies that react to their injuries
- [ ] Simple behaviour: back off, come back, fight back or run away
- [ ] Per-character personality (toughness, courage, fighting style)
- [ ] Every value tunable live through settings

**Later**
- [ ] Kicks, grabs, shoves and throws
- [ ] Crowds of characters walking around a map
- [ ] Characters reacting to each other (helping or fleeing)
- [ ] Switchable first- and third-person camera

## Getting started

Rophoric doesn't need Rojo or any other tool. The scripts go straight into Roblox Studio:

1. Download or clone this repository.
2. Open `MANIFEST.md`. It lists every script, where it goes in Studio and what type of script it is.
3. Create each script in Studio at that location, and paste in the file's contents.
4. Press **Play**. The test scene builds itself.

Controls and settings are documented below once the first version is out.

## Content warning

Rophoric includes **realistic blood, wounds and dismemberment**. On Roblox, an experience with this content needs a matching maturity rating: complete the experience's maturity questionnaire before publishing. Gore can be turned down or off in the settings.

## Repository layout

| Folder | What's in it |
|---|---|
| `src/` | The engine and test game. Folders mirror Roblox Studio's Explorer. |
| `docs/` | Design documents. `VISION.md` describes the goal in detail. |
| `reference/` | Reference recordings used to study how hit reactions should look. |
| `MANIFEST.md` | Where each script goes in Studio. |
| `TUNING.md` | What every setting does, and how to adjust the feel. |

## Inspiration and research

Rophoric is an independent project. It isn't affiliated with NaturalMotion, Rockstar Games or any of the games mentioned here, and it contains no code from them. It builds on published research on physically simulated characters, including:

- Zordan & Hodgins, *Motion Capture-Driven Simulations that Hit and React* (2002)
- Zordan et al., *Dynamic Response for Motion Capture Animation* (2005)
- Yin, Loken & van de Panne, *SIMBICON: Simple Biped Locomotion Control* (2007)
- Pratt et al., *Capture Point: A Step toward Humanoid Push Recovery* (2006)
- Geijtenbeek & Pronost, *Interactive Character Animation Using Simulated Physics* (2012)

Motion data and other third-party material, if used, are credited in `CREDITS.md`.

## License

Not chosen yet.
