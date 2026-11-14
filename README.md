# btc-cent-forecast-01 — entry

Forecasts Bitcoin's US-dollar price 240 minutes after each checkpoint, from
the 72 hours of 1-minute closes up to it. One program, `main.py` (Python 3.13,
standard library only), run once per case: it reads `/input/<id>.in` and
writes one guess to `/output/<id>.out`.

## The idea

Four-hour Bitcoin returns are nearly unpredictable, and the honest forecast
under a mean-absolute-error score is the conditional median of the price 4
hours out. The house baseline — guessing the checkpoint's own close — is
already most of the way there. The one small, robust signal found by
backtesting 35 days of 1-minute BTC closes (and the kit's practice cases) is
a short-horizon pullback: when the checkpoint close sits above the mean of
the last hour's closes, it tends to drift part-way back down over the next
four hours, and vice versa.

So the guess is the checkpoint close, pulled back by a tenth of its
deviation from the last 30 minutes' mean and a little over a tenth of its
deviation from the last hour's mean, with the move capped at ±0.5% so a
violent last hour can't produce a wild guess:

```
dev30 = log(last / mean(last 30 closes))
dev60 = log(last / mean(last 60 closes))
guess = last * exp(clip(-0.10 * dev30 - 0.12 * dev60, -0.005, +0.005))
```

No drift term and no long-horizon features: the final window's trend is
unknowable, trend-following loses to persistence out of sample, and the
long-horizon features carry no signal. In walk-forward blocks of twelve
4-hour-apart checkpoints — the final's shape — this beats persistence in
about 6 of 10 blocks by roughly half a percent of mean absolute error.

## Running it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

It is deterministic (pure arithmetic, no randomness, no network) and falls
back to the checkpoint's own close on any unexpected input, so a case stays
valid. Each run takes well under a second per case.

## License

MIT — see `LICENSE`.
