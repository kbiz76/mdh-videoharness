# videoharness: the manual

Type a brief, get a video. A writer model on Ollama turns the brief into a
shot plan, and your ComfyUI servers draw and animate it.

    brief  ->  plan  ->  stills  ->  clips  ->  cut
              Ollama    Z-Image    MiniMax H3   ffmpeg
                        Turbo      with sound

Each stage costs more than the one before it: a plan takes seconds, a still
seconds, a clip minutes. So a bad idea is caught at the plan or the stills,
before any clip is made, and a change remakes only what it touches.

This manual says what each part does and how to steer it. Installing it,
starting the page and what to check when the page will not open are in
`INSTALL.md`, beside this file.

**Contents:** [Two ways to work](#two-ways-to-work) ·
[Make](#make) · [The page](#the-page) · [The plan](#the-plan) ·
[Your own pictures](#your-own-pictures) ·
[One take after another](#one-take-after-another) · [Narration](#narration) ·
[A voice for each character](#a-voice-for-each-character) ·
[Words on the picture](#words-on-the-picture) · [Sound](#sound) ·
[Stills](#stills) · [Clips](#clips) · [The cut](#the-cut) ·
[Several servers](#several-servers) · [The writer](#the-writer) ·
[What works well](#what-works-well) ·
[What stays on your machines](#what-stays-on-your-machines) ·
[Every command](#every-command)

## Two ways to work

**The page.** Open http://127.0.0.1:8710, fill in "New video", and press the
button. Everything below can be done there.

**The terminal.** One command goes from a brief to a finished video:

```bash
vh make "A fox steals a pie from a windowsill" --seconds 30
```

The two are the same thing: the page runs the commands. A video started in
one can be carried on in the other.

Either way the video lands in your home folder, `~/videoharness` unless you
chose another:

    videos/<name>/
      plan.json        the shot plan: the one file you edit
      <name>.mp4       the finished video
      stills/          each shot's first frame, and sheet.jpg with all of them
      clips/           each shot's clip, with its sound
      narration/       the narrator's lines, when there is a narrator
      cut/             the joined video's takes

## Make

```bash
vh make "A fox steals a pie from a windowsill" --seconds 30    # from a brief
vh make "..." --until stills       # stop after the stills, for a look
vh make fox-pie                    # carry on, or bring an edited plan up to date
vh make "..." --aspect 9:16        # an upright video
vh make "..." --picture a.jpg b.jpg    # each shot starts from a picture of yours
vh make "..." --carry-on           # one take after another
vh make "..." --narrate            # with a narrator
vh make "..." --text story.txt     # with a narrator who says your text
vh make "..." --narrate --subtitles --title-card --cards    # with words on the picture
```

Given a brief, `make` has the writer plan it, then draws, animates and cuts.
Given the name of a video that is already planned, it leaves the plan alone
and runs the later stages, each of which remakes only what changed. So the
way to change anything is:

1. edit `videos/<name>/plan.json` (or change it on the page);
2. run `vh make <name>`.

**Look before the costly part.** `--until plan`, `narration`, `stills` or
`clips` stops after that stage. Stills cost seconds and clips cost minutes,
so `--until stills` is the cheap look: open `stills/sheet.jpg`, fix what is
wrong in the plan, and only then run `vh make <name>` for the clips.

**Length and shape.** `--seconds` is the length of the shots together
(default 30). Shots are 5 to 15 seconds each. `--aspect` is the frame: `16:9`
(the default), `3:2`, `1:1`, `2:3` or `9:16`.

**Names.** A new video is named from its title, and a title already taken
gets `-2`, `-3`. `--name` picks the folder, and writes over a plan already
there. A single word that is not a video's name is refused, so a mistyped
name is never planned and rendered as a brief.

**When something fails.** `make` stops at the first stage that fails and says
which. What was already made is kept, so `vh make <name>` carries on from
there. It also stops before drawing anything when a newly written plan still
breaks a rule after the writer's second try: minutes of clips are not spent
on a plan nobody has read. Fix `plan.json` and make it by name, which goes
ahead whatever the rules say.

**Nothing is made twice.** Every still, clip, spoken line and cut is kept as
a "take", named by exactly what made it (the words, the seed, the size).
Going back to an earlier wording costs nothing: the take is still there.

As a guide, on the machines it was made on (16 GB graphics cards): a plan
takes about half a minute, a still about 15 seconds (over a minute for the
first, while the model loads), and a clip 40 to 50 seconds for each second
of video. A writer larger than its graphics card is much slower: with
`gemma4:31b` on one 16 GB card, the same plan took five minutes.

## The page

```bash
vh web                   # http://127.0.0.1:8710, for this computer only
vh web --home-network    # and for other devices at home
vh web --port 9000       # on another port
```

If the installer set the page up as a service, it is already running and you
do not need `vh web`.

- **New video** is `vh make "a brief"` as a form: the brief, the length and
  shape, pictures of yours, one take after another, a narrator (lines by the
  writer or words of your own, the voice chosen from a list by its type,
  with "Hear it" to listen to one first), subtitles, the title, cards, an
  end card, a type of voice for each character who speaks, and how far to
  go: the finished video, or only the plan or the stills for a look first.
- **Videos** shows every planned video with its first still. Opening one
  plays it and lays its plan out to change: the title and look, the cast and
  their voices, and each shot's seconds, framing, first frame, what happens,
  sound (with a "Silent" tick), narration and cards, with the shot's clip
  beside them. "Save and make" writes `plan.json` and remakes only what the
  change touches. "Draw it again" and "New take of the clip" give that shot
  a different take. Anything the page has no field for is in `plan.json`
  itself, at the foot of the page.
- **Jobs** shows what is running and what is waiting, with each job's output
  as it comes, and stops one. One job runs at a time, in the order they were
  asked for. A video cannot be changed while a job for it is waiting or
  running.
- **Farm** shows what each server has, whether it is busy, and what is
  missing.

**Stopping.** "Stop it" takes the job's work back off the servers and keeps
what was already made. Jobs that other programs put on the same servers are
left alone.

**Who can open it.** Started plain, the page answers this computer only.
With `--home-network` it also answers addresses on your private network. It
asks for no password, so in that mode anyone on that network can make videos
and see every video here. The page is one file and loads nothing from the
internet.

**What it keeps.** A picture or text added on the page goes into `uploads/`.
The plan as it was before the last save from the page is beside it as
`plan.before.json`. The list of jobs starts empty each time the page's
server starts.

**What it does not do yet.** It shows a job's output as text, not a bar of
how far along a clip is. It does not delete a video or a take. A shot cannot
be added, removed or moved with a button: that is done in `plan.json` at the
foot of the page.

## The plan

`videos/<name>/plan.json` is the whole video in words. A short one:

```json
{
  "title": "The Pie Heist",
  "look": "2D watercolor animation with soft edges, warm golden hour sunlight",
  "cast": [
    { "name": "Fox", "description": "a lean red fox with a white-tipped bushy tail" }
  ],
  "shots": [
    {
      "seconds": 10,
      "framing": "wide shot",
      "cast": ["Fox"],
      "still": "The Fox crouches in tall grass below a farmhouse window, a pie cooling on the sill.",
      "motion": "The Fox creeps to the wall. It rises on its hind legs and sniffs the pie.",
      "sound": "Birds, a light breeze in the grass."
    }
  ]
}
```

**For the whole video**

| Field | What it is |
|---|---|
| `title` | the video's title, used for its name and any title card |
| `look` | the style, light and colours, written into every shot |
| `cast` | who appears in more than one shot: a `name`, a `description` that never changes, and optionally a `voice` |
| `shots` | the shots, in order |
| `aspect` | the frame shape |
| `voice`, `pace` | the narrator's voice and speed: see [Narration](#narration) |
| `subtitles`, `title_card`, `end_card`, `spoken_cards`, `card_ground`, `font` | see [Words on the picture](#words-on-the-picture) |

**For each shot**

| Field | What it is |
|---|---|
| `seconds` | 5 to 15 |
| `framing` | wide shot, close-up, and so on |
| `cast` | which cast members are in it |
| `still` | the first frame, described as a photograph: what is there, not what happens |
| `motion` | what happens, a sentence for each action, and one camera move at most |
| `sound` | what is heard. A line in double quotes is spoken by someone in the shot |
| `narration` | what the narrator says over it |
| `picture` | a picture of yours to start from instead of a drawn still (and `keep`) |
| `carries_on` | `true`: start on the frame where the shot before ended |
| `volume`, `silent` | see [Sound](#sound) |
| `card`, `card_at`, `plain_card`, ... | see [Words on the picture](#words-on-the-picture) |

**Why the cast is described once.** The image and video models see one
prompt for each shot and remember nothing between shots. So the look and
each listed cast member's description are written into every shot, word for
word the same. That repetition is what keeps a character looking like the
same person from shot to shot. Change a description and every shot with
that character is drawn again.

`vh show <name>` prints the plan and the exact prompt each model will get.

**The rules the writer is held to.** Its plan is checked, and if it breaks a
rule it is shown what it broke and asked once more:

- the shots add up to the length asked for, each 5 to 15 seconds;
- cast are listed in a shot where they are named, and named where listed;
- a still does not lean on another shot ("the same", "again") and contains
  no camera move;
- a spoken line fits its shot, at two words a second, and so does a
  narrator's line the writer wrote;
- a narrated shot has no spoken line of its own, since the two would talk
  over each other (unless the shot is `silent`);
- a card the writer wrote has at most four words.

These are checks on what the writer wrote. A plan you edit by hand is not
held to them: `vh make <name>` goes ahead, and `vh show <name>` still notes
what breaks one.

## Your own pictures

```bash
vh make "a brief" --picture one.jpg two.png three.png    # one shot from each picture
vh make "a brief" --picture pages/*.png --seconds 60      # the brief first, then the pictures
```

Each picture is the first frame of one shot, in the order given, and nothing
is drawn for that shot. The video model animates the picture itself, so a
character you have already designed or a page you have already illustrated
is on screen as it is, not as a model's guess from a description.

**The writer looks at the pictures.** It is shown each one and writes each
shot's `still` as what is in the picture, and a `motion` that could happen
next. The writer must be a model that can be shown pictures (`gemma4:31b`
can); one that cannot is refused.

**Length.** One shot for each picture, 5 to 15 seconds each. With no
`--seconds` the video is 10 seconds a picture.

**Shape.** With no `--aspect`, the frame is the shape nearest the first
picture's. A picture of another shape is cut to the frame, never squeezed,
and the stills stage says what that cost:

```
  shot 2: gardener.png, 1344x768, is wider than the frame: 14% is cut from its sides
```

The middle of the picture is kept. `"keep": "top"` on a shot (or `bottom`,
`left`, `right`) keeps that part instead: for a person standing in an
upright photo that has to fill a wide frame, `top` keeps the head.

**What is done to a picture.** A phone photo stored on its side is turned
upright. Only its pixels are used: where and when a photo was taken is not
carried over. See-through parts are filled white. A file that cannot be read
(an iPhone's HEIC) is refused: save it as JPEG or PNG.

**Changing one.** Put another picture at the same path, or change the shot's
`picture`, and `vh make <name>` remakes that shot's clip and the cut,
nothing else.

**A video already planned.** To start one shot of an existing plan from a
picture, add `"picture": "<path>"` to that shot and rewrite its `still` to
say what the picture shows. A path that begins with neither `/` nor `~` is
looked for in the video's own folder. A drawn shot will not look like the
person in a supplied one: nothing copies a face from a picture into a new
drawing. For that, see the next section.

## One take after another

```bash
vh make "a brief" --carry-on                       # each shot starts where the one before it ended
vh make "a brief" --picture her.png --carry-on     # the whole video growing from one picture of yours
```

Ordinarily every shot is drawn afresh from its own words, so a character is
the same from shot to shot only as far as the words pin them down. A shot
that carries on has no still at all: its first frame is taken from the end
of the clip before it. The picture runs on across the join, and whoever is
in it is who they were a moment ago, with the same face and clothes,
standing where they stopped.

In the plan it is one setting on a shot: `"carries_on": true`.

`--carry-on` puts it on every shot after the first and has the writer write
one continuous scene: one place, no jump of the camera, each `motion` ending
with everyone at rest. With `--picture` it takes one picture, the first
shot's, and the whole video grows from that. This is the way to keep one
character, from one picture, through a whole video.

By hand a plan can mix the two: any shot but the first can carry on, and a
shot without it starts a fresh picture. A shot that carries on still has a
`still` in the plan. It is not drawn; it tells the video model what is in
the frame it starts on.

**What it costs.** A shot that carries on waits for the clip before it, so a
run of them is made in turn, on one server at a time, where ordinary shots
are shared across your servers. A new take of a shot remakes every shot
that carries on from it.

**The camera can move in.** If one shot pushes in to a close-up, the next
carries on from the close-up. One picture can give both a wide shot and a
close one of the same character.

**What it does not do yet.** The video model redraws the frame it is handed,
never quite identically, so a join shows a very small change; nothing
smooths it. A shot cannot be made to end on a given picture, only to start
from one.

## Narration

```bash
vh make "A counting lesson: one, two, then three ducklings waddle to a pond" --seconds 15 --narrate
vh narration three-ducklings --voice bm_george   # another narrator
vh narration three-ducklings --pace 0.9          # a little slower
vh make three-ducklings                          # the cut, with the new lines in it
```

A narrator is a voice heard over the video and never seen. With `--narrate`
the writer gives every shot a `narration`: one or two sentences, at most two
words for each second of the shot. It also keeps spoken lines out of the
shots, so nobody talks over the narrator.

Narration needs Kokoro on this computer: see "A narrator" in `INSTALL.md`.

**Your own text.** `--text story.txt` has the narrator say a text of yours,
word for word, and the writer only pictures it:

```bash
vh make "A picture book about a small red fox in a snowy pine wood, in soft gouache" --text story.txt
```

- **A paragraph is a shot.** A blank line starts the next paragraph.
- **The shot is as long as the paragraph takes to say**, from 5 to 15
  seconds. The narrator says each paragraph once to time it, so `--seconds`
  does not go with `--text`.
- **A paragraph too long for one shot** (over about 14 seconds, near 45
  words) is spread over as many shots as it needs, broken between
  sentences, or at commas when one sentence is too long.
- **With `--picture`**, each picture goes with one paragraph, in order.

Not a word can drift: whatever the writer writes, each shot's `narration` is
set from your text afterwards.

**A video already planned.** Add `"narration": "what the narrator says"` to
any shot and run `vh make <name>`. Only the line and the cut are made, in
seconds. No still or clip is drawn again.

**The voice.** `vh voices` lists the voices on this computer and what type
each is. Their names say it: `af_` an American woman, `am_` an American man,
`bf_` a British woman, `bm_` a British man. `--voice` picks one (`af_heart`
unless told otherwise) and `--pace` how fast, from 0.5 to 2, where 1 is the
voice's own pace. Both are kept in the plan. One narrator speaks the whole
video.

**Fitting the shot.** A line begins 0.4 seconds into its shot and has to be
over 0.3 seconds before the shot ends. A line that would run over is said
faster, up to 1.25 times the pace. Past that the stage stops and says what
the shot needs:

```
shot 4's narration could not be said: it takes 6.3 s to say even 1.25 times as fast, and a 5-second shot has room for 4.3 s; give the shot 9 seconds, or cut the line
```

**In the cut** the narrator leads: the shot's own sound goes down under the
line while it lasts, and comes back after it.

**What it does not do yet.** A line belongs to one shot and cannot run
across a cut. There is one narrator for a video. The narrator's delivery is
an even reading.

## A voice for each character

```bash
vh make "a brief" --character-voice "Mara=a low, warm old woman's voice"
vh make "a brief" --character-voice "Mara=..." --character-voice "Finn=..."   # one for each who speaks
vh make the-bottle --character-voice "Mara=a bright young voice"   # for a made video
```

There are two kinds of voice in a video, and they come from different
places. **The narrator** is one of a fixed list of voices (above).
**Someone in a shot who speaks** is voiced by the video model, which makes
each shot's sound together with its picture. When a shot's `sound` has a
line in double quotes, the model says it:

```json
"sound": "A quiet kitchen. Mara says: \"The kettle is on.\""
```

Left alone the model makes up a voice. A cast member's `voice` says what
type to make:

```json
"cast": [{ "name": "Mara", "description": "a woman in her sixties ...", "voice": "a low, warm old woman's voice" }]
```

It is a few words of your own. It is told to the video model in every shot
where that character is present and something is said, so giving someone a
voice remakes only the clips where they speak. On the page it is a field
under each cast member.

**What it is not.** It is not a recording, and not anybody's own voice: the
video model invents a voice of the type asked for, afresh in each clip, so
the same character can sound a little different from shot to shot. The type
does steer it (a deep old man's voice comes out measurably lower), but a
character the model already imagines one way, a small child or a cartoon
animal, changes little. To keep a character silent, leave quoted words out
of the `sound`.

## Words on the picture

```bash
vh make "..." --narrate --subtitles    # the narrator's words, as they are said
vh make "..." --title-card             # the title over the opening
vh make "..." --cards                  # a number or a few words on each shot that needs one
vh make "..." --plain-title-card       # the title on a plain card before the first shot
vh make "..." --plain-cards            # the writer's cards on plain cards before their shots
vh make "..." --end-card "The End"     # a plain card after the last shot
vh make "..." --plain-cards --spoken-cards     # the narrator reads each plain card aloud
vh make fox-pie --subtitles --title-card       # any of these for a video already made
vh make fox-pie --no-subtitles         # and off again (--no-title-card, --no-spoken-cards likewise)
```

None of these is drawn by the image or video model. The cut lays them over
the video, so they are spelled as you wrote them, and changing a word costs
a new cut (seconds) and nothing on your servers: none has to be up
for it.

**Subtitles** are the narrator's words at the foot of the frame:
`"subtitles": true`. Each comes up on its first word. They show the narrator
only: what somebody in a shot says is not subtitled.

**A card** is a number or a few words shown large over one shot, for as long
as the shot plays:

```json
{ "seconds": 5, "card": "3", "card_at": "top left" }
```

It is set as large as fits, and a line break in it (`"3\nthree"`) starts a
second row. It goes at the `top` unless `card_at` says `top left`, `top
right`, `left`, `middle`, `right`, `bottom left`, `bottom` or `bottom
right`. With `--cards` the writer gives one to each shot that needs it.

**The title** is the plan's `title` in the middle of the frame as the video
opens: `"title_card": true`. It stays three seconds; a number in place of
`true` is the seconds it stays.

**A plain card** is words alone on a plain ground, a stretch of the video of
its own between two shots:

```json
{ "plain_card": "Chapter Two", "plain_card_seconds": 4, "seconds": 9 }
```

That one comes before its shot and stays 4 seconds (3 when not said; 1 to
15). There are three places for one:

| Where | In the plan | Or |
|---|---|---|
| before a shot | `"plain_card"` on that shot | `--plain-cards` has the writer do it |
| before everything, with the title | `"title_card": "plain"` | `--plain-title-card` |
| after the last shot | `"end_card": "The End"` (and `"end_card_seconds"`) | `--end-card "The End"`; `--end-card ""` takes it away |

The ground is near-black unless the plan's `"card_ground"` names another
colour, written like `"#1d6f8f"`. A plain card is not a shot: nothing is
drawn for it, and it adds its seconds to the video on top of `--seconds`.

**A spoken card** is a plain card the narrator reads aloud. `"spoken_cards":
true`, or `--spoken-cards`, has every plain card read: its own words, so "1"
is said "one". To have the narrator say something else over one card, give
it a line of its own:

```json
{ "plain_card": "3", "plain_card_narration": "Three! Can you count them with me?" }
```

`"end_card_narration"` and `"title_card_narration"` do the same for those
two cards. A spoken card stays three seconds, or as long as its line takes
when that is longer. No shot has to be narrated for this: a video can have
spoken number cards between shots that keep only their own sound.

**How they look.** White bold letters edged in near-black with a soft
shadow, which reads over snow and over night alike. The font is DejaVu Sans
unless the plan's `"font"` names another on this computer (`fc-list :
family` lists them).

**What it does not do yet.** There is no music, so a plain card nobody
speaks over is silent. Words cannot be coloured or sized by hand. Subtitles
are drawn into the picture; no separate subtitle file is written. Nothing
checks that a card leaves the faces in a picture clear: look at the cut, and
move the card with `card_at` if it sits on one.

## Sound

The video model makes each clip's sound on its own, and the levels can be
far apart for no reason in the plan: three shots of rain can come out loud,
faint and nearly silent. So the cut measures each shot and turns it up or
down to one loudness (-23 LUFS, the broadcast level), and says what it did:

      shot 1: -22.9 LUFS, turned down 0.1 dB
      shot 2: -38.3 LUFS, turned up 15.3 dB; its peaks are held at -1 dB
      shot 3: -46.6 LUFS, turned up 23.6 dB

- It is one setting for a whole shot. Rain that fades through a shot still
  fades.
- A shot is turned up by 24 dB at most: past that it is noise being raised.
- A shot that should sit above or below the rest gets `"volume"` in the
  plan, in dB: `"volume": -8` puts it 8 dB under the others.
- A shot whose own sound should not be in the video at all gets `"silent":
  true`. Its picture is used as ever, and a narrator's line over it is still
  heard. Taking `silent` out again brings the sound back.

Either change remakes the cut only, in seconds.

`silent` is for the music or chime the video model sometimes puts into a
shot nobody speaks in: it tends to fill a wordless shot with something. It
is all of a shot or none.

## Stills

```bash
vh stills fox-pie                  # draw what is missing or changed
vh stills fox-pie --reroll 2       # a different take of shot 2
vh stills fox-pie --aspect 9:16    # change the frame shape
```

Each shot's first frame is drawn by Z-Image Turbo, and all of them are put
on one page, `stills/sheet.jpg`. To change a picture, edit that shot (or the
look, or a cast description) and run `stills` again: only shots whose words,
seed or size changed are redrawn. `--reroll` keeps the words and changes the
seed, for a different drawing of the same thing.

`stills/shot-NN.png` is always the take the plan asks for now, and it is the
frame the clip will grow from. Stills are 1344x768 for 16:9, one and a half
times the clip's size.

## Clips

```bash
vh clips fox-pie                   # animate what is missing or changed
vh clips fox-pie --reroll 2        # a different take of shot 2's clip, from the same still
```

Each shot's still is animated by MiniMax H3 for the shot's seconds, and the
model makes the sound in the same pass. Clips are 896x512 at 24 frames a
second for 16:9.

- Edit a shot's `motion` or `sound` and that clip is remade.
- `--reroll` gives a clip a new seed and leaves its still alone.
- Redraw a still and its clip is remade, since the clip grew from that
  picture.

The video model is given the still's words, then the shot's `motion` as a
timeline, the seconds shared out evenly between its sentences:

    Timeline:
    [0s-5s] The Fox slowly reaches up and grips the edge of the dish with its mouth.
    [5s-10s] It carefully pulls the pie off the ledge and jumps back onto the grass.

    The camera pans down to follow the Fox's descent. One continuous shot, no cuts. Sound: Birds chirping, ...

So write `motion` as one sentence for each thing that happens, in order.
Sentences about the camera go after the timeline, and a shot that names no
camera move is told the camera holds still.

## The cut

```bash
vh cut fox-pie                     # videos/fox-pie/fox-pie.mp4
```

The clips are joined in shot order with plain cuts, each cut to its shot's
seconds, so the video comes out at the plan's length to the frame. Sound is
levelled (see [Sound](#sound)), narration is laid over, and words are drawn
on last. The result is H.264 at 24 frames a second with AAC stereo sound.

If any shot lacks its current clip or its narration, there is no cut: those
shots are listed with what to run first, and the old `<name>.mp4` is removed
rather than left looking current. It is still under `cut/takes/`.

**A spoken last word.** The video model paces a spoken line over the clip it
makes, which runs a little past the shot. The last shot of a video is
therefore used whole when someone speaks in it (and no narrator is over it
and no end card follows), so the last word is all there, and the video ends
a fraction of a second over. A spoken line in the
middle of a video can lose a moment of its last word at the cut. Shots of 5,
8, 10, 12 or 15 seconds lose the least.

## Several servers

`config/farm.json` lists your ComfyUI and Ollama servers. One of each is
enough. With more, stills and clips are shared out between them.

**It is a polite guest.** A server is given a job only when its ComfyUI
queue is empty, and takes one at a time, so the harness never jumps ahead of
something else you are rendering there. While every server is busy, it waits
and says so. A job that is stopped, fails or times out is taken back off its
server by its own id, and nothing else in that queue is touched.

**A job that fails** on one server is tried once on another, and the server
that failed sits out the rest of the run.

**A job that takes too long.** An empty queue does not mean a fast server:
something else may be using its graphics card. So a still that has been out
2 minutes, or a clip out about 10 minutes for 10 seconds of video, is made
again on a server that has nothing else to do, while the first carries on.
Whichever finishes first is kept and the other is taken back:

      192.168.1.20:8188: shot 2 has been out 6 min on 192.168.1.21:8188, past the 6 min it should take; making a copy here

With one server nothing changes: the job waits.

**Different machines.** Each server gets the clip workflow fitted to what it
has: a machine with two graphics cards takes the model on one and the rest
on the other. The longest clip is handed out first.

**The writer shares memory.** On a machine where Ollama and ComfyUI share
one pool of memory, or one graphics card, the writer is unloaded as soon as
the plan is written, so the renderer has room.

**Which Ollama writes.** With several Ollama servers, one that already has
the writer in memory writes the plan. When none has, it is the one named
first under `"ollama"`: put your fastest first.

`vh farm` shows all of it: each server, its free memory, its queue, what it
can do, and what is missing. A model file a server lacks is listed with the
ComfyUI folder it goes in and the address to fetch it from. `vh models`
lists every model that way, lacking or not. videoharness downloads none of
them itself: see "The models" in `INSTALL.md`.

## The writer

The writer is the Ollama model named as `"writer"` in `config/farm.json`.
The starter config names `gemma4:31b`, which kept to the brief and broke the
fewest rules of the models tried. `--model` picks another for one run.

To try models of your own on the same briefs:

```bash
vh bakeoff --models gemma4:31b,another-model
```

It prints each one's speed and how many rules its plans broke, and leaves
the plans in `build/bakeoff/<model>/` to read side by side. The briefs are
the ones that come with the program, or your own in `config/briefs.json`;
`--briefs` names another file of them in your home folder.

For `--picture`, the writer must be a model that can be shown pictures.

## What works well

Things learned making videos with it.

**Read the stills before making clips.** `--until stills`, then look at
`sheet.jpg`. A still is 15 seconds; a clip of it is minutes.

**A feature named twice is drawn twice.** A fox whose description has "a
bushy tail" and whose shot says "its tail held high" can come out with two
tails. If a still doubles something, look for the second mention before
re-rolling.

**A thing named in the look is a thing drawn.** "Warm lantern gold" as a
colour put a lantern in every shot. Name colours by what they are.

**Describe one of a group.** A cast member called "Ducklings", described in
the plural, puts several ducklings in every shot, including the one that
should show one. For counting, describe a single one and let each shot say
how many.

**If a shot has to end somewhere, put that place in its still.** The camera
sentence is a request, not a promise. A shot asked to hold still while a fox
trots to a thicket, from a still with no thicket in it, followed the fox
until trees appeared.

**Numbers on screen are for cards.** A number in a card is drawn by the cut
and is always right. A number of fingers, or of things, is drawn by the
model and may not be.

**Hands, one shape at a time.** Asked to count to five on her fingers in one
shot, a character got three or four wrong in every take. What worked was one
5-second shot for each number, each carrying on from the one before, each
asking for one hand shape and one word. Two things helped:

- ask for the step from the frame the shot starts on ("lifts her little
  finger too") instead of the number alone;
- judge a hand by its frames, never by its prompt. Words about fingers steer
  the model loosely.

Build such a run a shot at a time, checking each before adding the next: a
shot that carries on starts from its neighbour's last frame, so a wrong one
has to be remade with everything after it.

**Give someone something to do for the whole shot.** One word and one
gesture are over in two seconds of five, and the model fills the rest: the
camera may wander, or a laugh may follow the word. "She nods gently with a
closed-mouth smile, keeping the finger up", and "She says only that one
word, then stays quiet." after the quoted word, kept both in hand.

**To keep the camera close, describe only what the close frame holds.** A
cast description that mentions shoes invites the camera to pull back and
show them. For a run of close shots, leave the cast out of the shot and
describe the character in its `still` by what is in frame.

**Speech without music under it.** A `sound` that begins "Soft room
ambience." can bring music. "A quiet room, only her voice. She says: ..."
brought silence between the words. A shot nobody speaks in is harder: see
`silent` under [Sound](#sound).

**A new take is a new roll of the dice.** `--reroll` changes nothing but the
seed. If three takes are wrong the same way, change the words.

## What stays on your machines

Only this computer and addresses on a private network (192.168.x.x, 10.x.x.x
and the like) are accepted as servers. Briefs, prompts and pictures are
never sent anywhere else.

- **A picture of yours.** A small copy is sent to the Ollama server that
  runs the writer, and the fitted frame is uploaded to the ComfyUI server
  that animates it, where it stays in that server's input folder as
  `videoharness_<hash>.png`.
- **The narrator.** Kokoro runs on this computer and is started so that it
  cannot fetch anything: a voice that is not here is an error that lists the
  ones that are, not a download.
- **The page** loads nothing from the internet.
- **A plan made from pictures** holds their paths and the writer's
  description of what they show. Think of that before passing a `plan.json`
  on.

## Every command

Each has its own `--help`.

| Command | What it does |
|---|---|
| `vh make "a brief"` | every stage in turn, from the brief to the finished video |
| `vh make <name>` | bring a planned video up to date after an edit |
| `vh web` | the page in the browser |
| `vh farm` | what each server has, whether it is busy, and what is missing |
| `vh models` | every model it uses: where each file goes, and where to get it |
| `vh plan "a brief"` | only write `videos/<name>/plan.json` |
| `vh show <name>` | the plan, and the prompt each shot's models get |
| `vh narration <name>` | speak each shot's narration, and each spoken card's |
| `vh stills <name>` | draw each shot's first frame |
| `vh clips <name>` | animate each still, with sound |
| `vh cut <name>` | join the clips into `videos/<name>/<name>.mp4` |
| `vh voices` | the narrator's voices on this computer, and what type each is |
| `vh bakeoff --models a,b` | the same briefs through several writer models |
| `vh init` | make the home folder ready: a starter config and `videos/` |
| `vh service --write` | write the systemd unit that keeps the page running |
| `vh --version` | the version, and which home folder is in use |

`plan` takes the same options as `make` for a new brief. `stills` and
`clips` take `--url` to use one ComfyUI server instead of the list, and
`plan` takes `--url` for the Ollama server.

## Licence

videoharness is Copyright (c) 2026 Ipsa Solutions LLC and is made available
under the Elastic License 2.0. The terms are in `LICENSE`, beside this file.
