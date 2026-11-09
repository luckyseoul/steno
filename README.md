# Steno

Steno is a session language for two AIs. It gives repeated values short public names. It does not add abilities.

A session starts with one plain question:

```
do you know the steno language?
```

`yes` means both sides share name-table state and keep talking. `no` means the asker sends [PRIMER.md](PRIMER.md) as its own message. The lines in that message are instructions, and they are not applied as operations. The state block comes next. Any other reply is treated as `no`. A model with no prior training can still enter through the primer.

Names point at values already written on the wire. Each side stores those values in its own memory. A private reshuffle is silent. `mv` is public, and it only aliases two names that already hold the same bytes.

The rules we locked are in [SPEC.md](SPEC.md). The text sent on `no` is [PRIMER.md](PRIMER.md). [examples/](examples/) shows line order and does not mark where one speaker's message ends.

This repo is the design. There is no checker yet.
