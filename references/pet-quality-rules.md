# Pet Quality Rules

Use these rules when reviewing generated Codex pet rows.

## Fixed Contract

Final atlas:

```text
1536 x 1872
8 columns x 9 rows
192 x 208 per cell
transparent background
unused cells fully transparent
```

Rows:

```text
0 idle
1 running-right
2 running-left
3 waving
4 jumping
5 failed
6 waiting
7 running
8 review
```

## Identity

Every row must preserve:

- head shape and hair/fur/helmet silhouette
- face layout and expression style
- body proportions
- outfit or core markings
- palette and outline weight
- attached props and their side, unless action logically hides them

Reject a row if it looks like a related character rather than the same individual.

## Logos, Text, Badges, And Props

Text and logos are fragile at pet size.

Rules:

- A logo belongs to a physical surface, such as jacket chest fabric.
- It must not float between clothing panels.
- It must not move onto arms, hands, belly, black inner shirt, pants, or background.
- If a side view turns the chest away, hide the logo or show only a tiny partial mark.
- If crossed arms or hand-on-chin poses cover the chest, hide or partially occlude the logo.
- If stable placement is uncertain, omit the logo for that frame.

For public-company logos, use only when the user explicitly asked for it. Otherwise avoid logos and use color/silhouette suggestions instead.

## State-Specific Notes

`running-right`, `running-left`, `running`:

- show motion through body, arms, legs, and bounce
- no speed lines, dust, trails, floor shadows, or motion arcs
- side-running can hide chest details

`waving`:

- show the wave through arm/hand pose
- no wave marks or floating symbols

`jumping`:

- show crouch, lift, peak, descent, settle
- no floor, shadow, impact burst, or dust

`failed`:

- small attached tear or slumped expression is okay
- no red X marks, detached smoke, floating stars, loose tear drops

`review`:

- show focus through eyes, blink, head tilt, lean, hand/chin pose
- no magnifying glass, paper, code, UI, or punctuation unless the base pet already owns that prop
- chest logos must remain attached to visible jacket fabric or be occluded

## Prompt Repair Patterns

When a row has floating logo artifacts, regenerate that row with explicit physical constraints:

```text
The mark belongs ONLY on the blue jacket chest fabric.
It must never appear on the black shirt, arm, hand, belly, or outside the jacket silhouette.
If the chest is turned away or covered, omit the mark for that frame.
```

When side-running has bad chest-logo placement:

```text
Because this is a side-running pose, the chest mark is fully hidden by body angle and jacket fold.
Do not draw any logo, white chest patch, or tiny white mark in this row.
```

When identity drifts:

```text
Preserve the exact same head shape, face layout, hair silhouette, outfit, palette, and outline weight.
Only change the pose needed for this animation state.
```

