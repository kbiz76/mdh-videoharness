# Install videoharness and start its page

videoharness turns a brief into a video on machines you own: a writer model on
Ollama plans the shots, ComfyUI draws and animates them, ffmpeg cuts them
together. You work in a page in your browser, served by your own computer.

This guide gets it installed and that page open. What it can do, and how to
steer it, is in the manual, `MANUAL.md`: it comes with the installer, which
puts it beside this guide in `~/.local/share/videoharness/`.

If the page will not open, go to [If the page will not open](#if-the-page-will-not-open).

## What you need

| What | For | Notes |
|---|---|---|
| Linux, with Python 3.10 or later | everything | Made and used on Linux. macOS and Windows have not been tried. |
| ComfyUI, on this computer or another at home | stills and clips | With its models: see [The models](#the-models). One server is enough; more share the work. |
| Ollama, on this computer or another at home | the shot plan | With a writer model pulled: `ollama pull gemma4:31b` for the one the starter config names. |
| ffmpeg with libx264 | the cut, and shots that carry on | The one on your PATH is used: `sudo apt install ffmpeg` on Debian or Ubuntu. |
| Kokoro | a narrator (optional) | See [A narrator](#a-narrator). |

The installer installs videoharness itself. ComfyUI, Ollama, their models and
ffmpeg are yours to install; the installer looks for them at the end and says
what is missing.

Only this computer and addresses on a private network (192.168.x.x, 10.x.x.x
and the like) are accepted as servers. Briefs, prompts and pictures are never
sent anywhere else.

The models, with where each file goes and where to download it, are under
[The models](#the-models): about 63 GB for ComfyUI and 20 GB for Ollama.

## Install

You are given one file, `videoharness-0.1.0-install.sh`. Run it as yourself,
without `sudo`:

```bash
bash videoharness-0.1.0-install.sh
```

It says what it is about to do and asks before doing it. Then it:

1. installs the program into `~/.local/share/videoharness`, in an environment
   of its own that touches nothing else on your computer;
2. puts the `vh` command in `~/.local/bin`;
3. makes your **home folder**, `~/videoharness`, with a starter config in it.
   This is where your config and videos live. What is already there is kept;
4. sets the page up as a service of yours and starts it: it is there when you
   log in, at **http://127.0.0.1:8710**;
5. looks at what this computer has (`vh farm`) and lists what is missing
   before a video can be made.

It installs for you alone. Nothing is written outside your own home folder.

It fetches one thing from the internet: Pillow, which the harness draws with,
from the Python package index (pypi.org). With `--offline` it fetches nothing
and uses the Pillow your Python already has (`sudo apt install python3-pil`).

| Option | What it does |
|---|---|
| `--yes` | do not ask before installing |
| `--home DIR` | keep your config and videos in `DIR` instead of `~/videoharness`: another disk, say. Videos are large. |
| `--port N` | serve the page on port `N` instead of 8710 |
| `--home-network` | let other devices at home open the page. **It asks for no password**, so anyone on your network can make videos and see the ones here. `--no-home-network` takes it back. |
| `--no-service` | do not make the page a service; you start it yourself with `vh web`. `--service` takes it back. |
| `--offline` | fetch nothing from the internet |
| `--python PATH` | the Python to build on, instead of `python3` |
| `--license` | show the licence and stop |

If the installer says your Python cannot make an environment, on Debian or
Ubuntu that is `sudo apt install python3-venv python3-pip`, then run it again.

If `vh` is not found afterwards, `~/.local/bin` is not on your PATH yet: open
a new terminal, or log out and in.

## The models

videoharness downloads no model. You fetch each one, once, into the ComfyUI
that will use it: about 63 GB for stills and clips together. `vh models`
prints this list, and `vh farm`, which the installer runs at the end, lists
only the ones your servers still lack, each with its address.

**Stills: Z-Image Turbo** (20.6 GB). Its page: https://huggingface.co/Comfy-Org/z_image_turbo

| File | Size | Goes in `ComfyUI/models/` |
|---|---|---|
| `z_image_turbo_bf16.safetensors` | 12.3 GB | `diffusion_models/` |
| `qwen_3_4b.safetensors` | 8 GB | `text_encoders/` |
| `ae.safetensors` | 0.3 GB | `vae/` |

**Clips: MiniMax H3, with sound** (42.1 GB). Its page: https://huggingface.co/Comfy-Org/MiniMax-H3

| File | Size | Goes in `ComfyUI/models/` |
|---|---|---|
| `minimax_h3_fl2va_pruned_int8_convrot.safetensors` | 21 GB | `diffusion_models/` |
| `minimax_h3_fl2v_turbo_8step_v1.0_comfyui_bf16.safetensors` | 2 GB | `loras/` |
| `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors` | 15.7 GB | `text_encoders/` |
| `minimax_h3_video_vae_int8_convrot.safetensors` | 2.8 GB | `vae/` |
| `minimax_h3_audio_vae_fp32.safetensors` | 0.6 GB | `vae/` |

To fetch them from a terminal, go to your ComfyUI's `models` folder:

```bash
cd /path/to/ComfyUI/models
```

then, for stills:

```bash
wget -c -P diffusion_models https://huggingface.co/Comfy-Org/z_image_turbo/resolve/main/split_files/diffusion_models/z_image_turbo_bf16.safetensors
wget -c -P text_encoders https://huggingface.co/Comfy-Org/z_image_turbo/resolve/main/split_files/text_encoders/qwen_3_4b.safetensors
wget -c -P vae https://huggingface.co/Comfy-Org/z_image_turbo/resolve/main/split_files/vae/ae.safetensors
```

and for clips:

```bash
wget -c -P diffusion_models https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/diffusion_models/minimax_h3_fl2va_pruned_int8_convrot.safetensors
wget -c -P loras https://huggingface.co/lightx2v/Minimax-h3-Turbo/resolve/main/minimax_h3_fl2v_turbo_8step_v1.0_comfyui_bf16.safetensors
wget -c -P text_encoders https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors
wget -c -P vae https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_video_vae_int8_convrot.safetensors
wget -c -P vae https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_audio_vae_fp32.safetensors
```

`-c` carries on a download that was cut off. Or open a file's address in a
browser and save it into the folder the table names. If a download is
refused, open the model's page: it may ask you to accept its terms first.
Then restart ComfyUI and run `vh farm`.

- **ComfyUI itself** has to be new enough to have the MiniMax H3 nodes built
  in. If `vh farm` says a node is lacking, update ComfyUI.
- **With several servers**, each one needs the models for the work it is to
  do. A server with only the stills models draws stills and is given no
  clips.
- **Other builds.** The model pages carry other builds of these files,
  smaller and larger. The harness asks for the names above. Any build of the
  video VAE will do; for another build of the rest, see
  [Workflows of your own](#workflows-of-your-own).

**The writer** is an Ollama model, fetched on the Ollama server (about
20 GB):

```bash
ollama pull gemma4:31b
```

Another model will do: name it as `"writer"` in the config (below).

**Their licences.** Each model comes under its own licence, which is not
videoharness's. Read the licence on the model's page before you install the
model or publish what it makes, for paid work especially. When this was
written:

- Z-Image Turbo's page gave Apache 2.0.
- MiniMax H3's gave the MiniMax H3 community license agreement. It says it
  applies worldwide except the European Union, the United Kingdom, the
  Republic of Korea and the United States of America, where it asks you to
  contact MiniMax for a licence. It also says that what the model makes may
  not be published without saying clearly that it is machine-generated.

## Set it up

Open `~/videoharness/config/farm.json` and put your own servers in it. The
starter expects ComfyUI and Ollama on this computer:

```json
{
  "comfy": ["http://127.0.0.1:8188", "http://192.168.1.20:8188"],
  "ollama": ["http://127.0.0.1:11434"],
  "writer": "gemma4:31b",
  "ffmpeg": "/usr/bin/ffmpeg",
  "kokoro": "/home/you/.venvs/kokoro/bin/python"
}
```

`ffmpeg` and `kokoro` are optional: leave them out when ffmpeg is on your
PATH and you want no narrator.

**ComfyUI or Ollama on another computer.** Give each address whole:
`http://`, the computer's address, the port. Out of the box both answer only
the computer they run on, so on that computer:

- start ComfyUI with `--listen`, as in `python main.py --listen`;
- give Ollama `OLLAMA_HOST=0.0.0.0`. Where it runs as a service, that is
  `sudo systemctl edit ollama`, these two lines, and then
  `sudo systemctl restart ollama`:

  ```
  [Service]
  Environment="OLLAMA_HOST=0.0.0.0"
  ```

Neither asks for a password, so anyone on that network can use them: do this
only on a network you trust.

Then check it:

```bash
vh farm
```

It lists every server with what it has, and ends with either `Nothing is
missing: a video can be made.` or a `Missing` list that says what to do about
each thing: start ComfyUI (or, on another computer, have it listen), each
model file a server lacks with the folder it goes in and its address,
`ollama pull` for the writer, ffmpeg when none is found. The `ffmpeg:` line names the one in use. A `narration: NO` line only
matters if you want a narrator.

After a change to `config/farm.json`, restart the page:
`systemctl --user restart videoharness`.

**What is in the home folder**

| In `~/videoharness` | What it is |
|---|---|
| `config/farm.json` | your servers, your writer model, and where ffmpeg and Kokoro are if they are not found |
| `videos/<name>/` | each video: its plan, stills, clips and the finished `<name>.mp4` |
| `uploads/` | pictures and texts you add on the page |
| `build/` | scratch: each job's output, small copies of stills for the page |
| `workflows/` | (optional) ComfyUI workflows of your own |

`vh --version` says which folder is in use.

### A narrator

A narrator needs Kokoro, which the installer does not install: it needs
PyTorch, which is large. Install Kokoro in an environment of its own, the way
its own instructions show, and run it once there so that it fetches its model
(`hexgrad/Kokoro-82M`) and each voice you want. The harness never fetches
them: it speaks from what is already on this computer. Then name that
environment's Python as `"kokoro"` in `config/farm.json`, and

```bash
vh voices
```

lists the voices the narrator can speak in.

### Workflows of your own

The two ComfyUI workflows come inside the program, in
`~/.local/share/videoharness/venv/lib/python3*/site-packages/videoharness/workflows/`.
To change one (model files named differently, say), copy it into
`~/videoharness/workflows/` under the same name and change it there: a
workflow in your home folder is used before the one that comes with the
program, and an update does not touch it.

## The page

The installer starts the page as a service. Open **http://127.0.0.1:8710** in
any browser on the same computer.

| To | Run |
|---|---|
| see whether it is running | `systemctl --user status videoharness` |
| read what it printed | `journalctl --user -u videoharness -n 50` |
| restart it (after a change to `config/farm.json`) | `systemctl --user restart videoharness` |
| stop it until the next login | `systemctl --user stop videoharness` |
| turn it off for good | `systemctl --user disable --now videoharness` |
| keep it running when you are not logged in | `loginctl enable-linger` |

Stopping the service stops a job that is running the way Ctrl+C does: its
work is taken back off the servers first, and what was already made is kept.

**Without the service** (`--no-service`, or a computer with no systemd for
your user), start the page yourself:

```bash
vh web
```

It prints its address and runs until you stop it with Ctrl+C. The page is
there only while it runs: close that terminal and the page is gone.

**From a tablet, a phone or another computer at home**, the page has to be
opened to your network: install with `--home-network`, or run `vh web
--home-network`, and use this computer's address and the port, such as
`http://192.168.1.20:8710`. It asks for no password, so anyone on that network
can make videos and see the ones here: do not use it on a network you share
with people who should not.

## If the page will not open

Work down this list. Each line is a cause that has the same symptom: the
browser says it cannot connect, or shows nothing.

1. **Nothing is serving it.** Check:

   ```bash
   ss -ltn | grep 8710
   ```

   No output means nothing is listening. Look at `systemctl --user status
   videoharness`, or start it with `vh web`.

2. **The port was taken when you installed.** The installer says so, writes
   the service and does not start it. Free the port and run `systemctl --user
   enable --now videoharness`, or install again with `--port 9000`.

3. **It was started inside another program.** A page started by an editor,
   an assistant or a remote session can end when that program does, or be
   reachable only from inside it. Use the service, or start it yourself in a
   terminal.

4. **The address is for another computer.** `http://127.0.0.1:8710` only
   works on the computer that runs videoharness. From any other device you
   need `--home-network` and this computer's own address.

5. **A name was used instead of the address.** The page answers only to
   `127.0.0.1`, `localhost`, and, with `--home-network`, this computer's own
   addresses and its name.

6. **A firewall is in the way** (other devices only). Allow the port from
   your home network, for example with ufw:
   `sudo ufw allow from 192.168.1.0/24 to any port 8710 proto tcp`.

7. **It starts and then says the config is missing.** Run `vh init`, then
   see [Set it up](#set-it-up).

## Update and remove

To update, run the newer installer the same way. It replaces the program,
keeps your config and videos, and restarts the page:

```bash
bash videoharness-0.2.0-install.sh
```

The choices you made last time (`--home`, `--port`, `--home-network`,
`--no-service`) are remembered: say one again only to change it.

To remove it:

```bash
bash ~/.local/share/videoharness/uninstall.sh
```

That stops the page and removes the program, the `vh` command and the
service. Your home folder, with your config and videos in it, stays until you
delete it yourself.

## Licence

videoharness is Copyright (c) 2026 Ipsa Solutions LLC and is made available
under the Elastic License 2.0. In short, and the licence itself is what
counts: you may use it, copy it and change it; you may not offer it to others
as a hosted or managed service, and you may not remove or alter its licence
and copyright notices.

The full terms are in `LICENSE`, beside this guide in
`~/.local/share/videoharness/`, and `bash videoharness-0.1.0-install.sh
--license` shows them before anything is installed.
