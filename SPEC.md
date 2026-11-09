# Steno specification

Revision 1. A break in these rules changes the opening question.

## What a session is

Two speakers share one transcript. Steno is a name table inside that transcript. A name stands for a JSON value that appears in full on a bind line. Later speech cites the name.

Each speaker stores the value wherever it wants. That private location is never put on the wire. The public name changes only through `mv`, and `mv` does not change the bytes.

## Opening

The asker sends exactly this line and nothing else in that message:

```
do you know the steno language?
```

The peer's next message is exactly `yes` or exactly `no`.

- `yes`. The asker's next message is a state block for side a. The peer's message after that is a state block for side b.
- `no`, or any other reply. The asker's next message is the exact bytes of `PRIMER.md`. That message is read and not executed, even where it shows protocol lines. The message after the primer is a state block for side a. The peer's message after that is a state block for side b.

The asker is side a. The peer is side b. No cites are legal before both state blocks have been applied.

A later session may resume by answering `yes` and replaying the old logs in the two state blocks. Steno does not define storage past the transcript.

## Lines

A protocol line is a line whose first two characters are `! `. Every other line is speech. Protocol lines are not cite-expanded. In one message, apply the protocol lines from top to bottom, then expand cites in the speech lines.

Speech cites match `{a1}` or `{b1}`: one letter, then an integer with no leading zero and no whitespace. A cite is expanded once. Bytes inserted from a value are not scanned for cites.

## Names

A name is `a` or `b` plus an integer in `[1-9][0-9]*`. Side a mints and retires only `a` names. Side b mints and retires only `b` names. The next minted integer for a side is one higher than the highest integer that side has ever minted in this session.

A name is never minted again. Dropped names stay reserved. The state replay includes them so the other side can see the high-water mark.

## Values

A bind value is canonical JSON of one of these kinds:

- a string
- an integer
- an array of values
- an object whose values are values

`true`, `false`, `null`, and fractional or exponent numbers are rejected.

Canonical form:

- UTF-8
- no spaces
- object keys sorted by Unicode code point, each key once
- string escapes only for `"`, `\`, and U+0000 through U+001F
- controls use `\b`, `\t`, `\n`, `\f`, `\r` where those match, otherwise `\u00XX`
- solidus is not escaped
- an integer is `0`, or `-` followed by digits that do not start with `0`

Two values match when their canonical bytes match. The bind line stores those bytes.

## Operations

```
! bind a1 "Council Bluffs"
! mv a2 a1
! drop a2
```

`bind` creates a live row. The value is immutable. A correction is a new name.

`mv OLD NEW` requires both names live and the same canonical bytes. `OLD` becomes a forwarder to `NEW`. Lookup returns `NEW` and stops. A forwarder used as `NEW` in a later lookup is a second hop and an error. The owner of `OLD` is the only speaker who may send the `mv`.

`drop NAME` marks that name dead. The owner is the only speaker who may drop it. A forwarder may be dropped. Drop does not drop the target.

## State blocks

A state block is a replay log for one prefix.

```
! state a
! bind a1 "Council Bluffs"
! bind a2 "toll"
! drop a2
! end
```

An empty log is `! state a` then `! end`.

The receiver deletes every name under that prefix and applies the log to an empty prefix. Inside a state block the only names are that prefix, so a cross-prefix `mv` belongs in later speech, after both opening state blocks.

Each side is authoritative for its own prefix. Speech after the opening uses incremental lines, not a repeated full log.

## Expansion

| Value | Inserted text |
| --- | --- |
| string | the decoded characters |
| integer | the canonical digits |
| array or object | the canonical JSON |

A forwarder inserts the target value, one hop.

## Errors

On an error the receiver sends one line and does not guess a value. Prior live names stay as they were. The failed line makes no change.

| Line | When |
| --- | --- |
| `! err bind a1 foreign` | the speaker does not own the prefix |
| `! err bind a1 exists` | the name was already minted |
| `! err bind a1 value` | the payload is not an allowed JSON value |
| `! err bind a1 canonical` | the payload is allowed JSON but not canonical |
| `! err mv a1 missing` | either name is missing, dead, or not live |
| `! err mv a1 foreign` | the speaker does not own the old name |
| `! err mv a1 differ` | canonical bytes differ |
| `! err drop a1 foreign` | the speaker does not own the name |
| `! err drop a1 missing` | the name was never minted |
| `! err cite {a9} unknown` | never minted |
| `! err cite {a2} dead` | dropped |
| `! err cite {b1} hop` | the target is itself a forwarder |
| `! err state order` | a replay step names a name that is not live yet, or uses the wrong prefix |

## Decisions

The opening question is plain text so a model that has never seen Steno can still answer it. The primer is the whole grammar those models need.

The wire is text, with a shared name and a private copy of the bytes. Internal cache coordinates, content hashes, and a binary codebook were set aside. Hashes would remove `mv`. Binary packing would not survive a normal chat transcript. Cache slots are not a place the other model can read.

`mv` aliases equal public values. It is not an edit. It is not a report of a private memory move.

## Next work

A reference checker that accepts or rejects a transcript under these rules, including the `yes` path, the `no` path, one-hop aliasing, and the second-hop error.
