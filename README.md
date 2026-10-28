# deixis robots

**Tell any robot where.** Point and say "here", "that one", "this big", and a robot acts on
the place you meant. This repo brings together the core and the three robots it drives, so one
clone gets everything.

Entry for the Nebius x NVIDIA Global AI Hackathon, Physical AI track.

## Get everything

```
git clone --recurse-submodules https://github.com/rahuljeyaraj/deixis-robots.git
cd deixis-robots
```

If you cloned without `--recurse-submodules`, run `git submodule update --init`.

## What is in here

| Folder | What it is | Licence |
|---|---|---|
| [`deixis`](https://github.com/rahuljeyaraj/deixis) | The core. Turns words and pointing into a place in the robot's own frame, with how sure it is, or one question when it is not sure. Pure code: no models, no camera, no robot. | MIT |
| [`deixis-doodle`](https://github.com/rahuljeyaraj/deixis-doodle) | The app the plotter and the vacuum share (camera, hand and feet tracking, speech, the model), and the pen plotter: a converted Ender 3 that draws what you asked, where you pointed, at the size you showed. | MIT |
| [`deixis-web-demo`](https://github.com/rahuljeyaraj/deixis-web-demo) | A virtual SO-101 robot arm in the browser, with a red, a green and a blue block. Point with a fingertip in front of the webcam (or the mouse) and say "put that here". | MIT |
| [`deixis-vacuum`](https://github.com/rahuljeyaraj/deixis-vacuum) | The robot vacuum's driver: sends an off-the-shelf ECOVACS DEEBOT to clean a box on its map, over MCP. Its own repo because it uses `deebot-client`, which is GPL-3.0. | GPL-3.0 |

The folders sit side by side on purpose: `deixis-doodle` and `deixis-web-demo` load the core
from `../deixis`.

All three robots reach the core the same way, over MCP. The vacuum goes through the
`deixis-doodle` app, which calls `deixis-vacuum` as its robot.

## Set-up

You need [uv](https://docs.astral.sh/uv/) and Python 3.11 or 3.12. The web demo also needs
Node 22 or newer. Each folder's own README has the full steps.

**The web demo** (the easiest way to try it; no robot needed):

```powershell
cd deixis-web-demo
copy .env.example .env           # then put a Nebius Token Factory key in .env
uv sync
uv run webdemo                   # the server on http://127.0.0.1:8000
cd web
npm install
npm run dev                      # the page on http://localhost:5173
```

**The core:**

```powershell
cd deixis
uv sync
uv run pytest
```

**The plotter and the vacuum app:**

```powershell
cd deixis-doodle
uv sync --all-extras
uv run pytest
```

**The vacuum's driver:** see [`deixis-vacuum/README.md`](https://github.com/rahuljeyaraj/deixis-vacuum#set-up)
(it needs an ECOVACS app login).

## NVIDIA models we use

| Model | What it does | Where it runs |
|---|---|---|
| **Nemotron 3.5 Lightning** | Turns the sentence into tool calls. It never writes a coordinate: it points at words ("here" is word 3), and the core turns them into millimetres. | Nebius Token Factory |
| **Parakeet TDT 0.6B v2** | Speech to words, with the time each word was said, so "here" can be matched to where the hand was at that moment. | On the robot's computer (CPU); the web demo server |
| **Cosmos 3 Nano** | Reads what is already drawn on the paper, so "next to the house" has something to match. Built into the plotter, but not in any scored run. | Nebius AI Cloud Serverless Endpoint |
| **SANA 1600M** | Made the plotter's drawings in advance: 852 pictures, from which a person picked the ones we use. | Nebius AI Cloud Serverless Job |

Nemotron and Parakeet run live while you talk. Cosmos and SANA do not.

**Why Lightning over Nemotron 3 Nano:** Lightning gives all of a sentence's calls in one reply,
so "a house here and a tree there" keeps the tree. On our 61-command test it got 52 right,
against Nano's 43, at the same price.

## Where Nebius Token Factory helped

- **Every command goes to Token Factory.** One key and the OpenAI SDK, the same for all three
  robots. Each command is one short call, so paying per token suits a robot: there is no
  machine to keep running.
- **Cheap to run:** at list price the arm's prompt comes to about **$0.34 per 1,000 commands**,
  the plotter's longer one about $0.50.
- **Cheap to choose a model:** comparing Nemotron models on our test set, in seven runs plus
  smaller checks, cost **$0.22** of tokens.

## Other Nebius tools

- **Serverless Endpoints** run Cosmos, because Token Factory has no NVIDIA vision model.
- **Serverless Jobs** ran SANA once as a batch: 852 drawings in about 24 minutes for about
  $0.72, on one RTX PRO 6000.

## Licence

This repo: MIT, see [LICENSE](LICENSE). Each folder keeps its own licence (table above).
