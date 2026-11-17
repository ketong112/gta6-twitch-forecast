# gta6-twitch-forecast

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/ketong112/gta6-twitch-forecast/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/ketong112/gta6-twitch-forecast/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

Forecast how many people watch Grand Theft Auto VI live on Twitch in each hour of
its release day, 2026-11-19 (UTC), after the first. A Node 22 program, standard
library only, no network: it reads one case file per run and writes one forecast
file with five quantiles (5/25/50/75/95%) of each hour's viewer count, hours 2-24.

## How it works

A launch day on Twitch is not flat: the count dips in the small hours, jumps when
the United States unlocks (05:00 UTC for GTA VI, one nationwide unlock at midnight
Eastern), then climbs through the US day and the European evening to a peak in
hours 19-22. This program learns that shape from the case's own comparables — 42
past big game launches with hourly Twitch counts:

- Each comparable's release-day curve is reduced to log-ratios,
  `ln(viewers[h] / viewers[1])`, so the shape is scale-free.
- The pool is filtered to fair analogues for a console-only, rolling-midnight,
  no-early-access launch: early-access-contaminated launches are out (their hour-1
  print is already inflated), simultaneous-global launches are out (no US-unlock
  jump to learn from), weak analogues are out, and GTA VI's own pre-release
  history is out (it is not a launch).
- Each comparable is weighted by a Gaussian kernel on log hour-1 print (sigma 1.0),
  centred on the case's own hour-1 print: launches that started the day at a
  similar size count most. A small hour-1 print means the big markets had not
  unlocked yet, so the day ramps 5-100x; a huge one means they had, so it ramps
  only 1-2x.
- For each hour, the five forecast quantiles are the weighted quantiles of that
  hour's log-ratio across the pool, scaled back by the hour-1 print.

Validated on the four practice cases against the historical counts (SullyGnome,
the same source the contest scores against): mean pinball loss **0.200**, against
the house baseline's **0.387** (Wolverine 0.290, Red Dead Redemption 2 0.163,
God of War Ragnarok 0.132, Marvel's Spider-Man 2 0.215).

## Run it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out node main.ts
```

Reads every `<id>.in` in the input directory and writes `<id>.out` for each.
Deterministic: two runs write the same bytes. No network, no dependencies.
