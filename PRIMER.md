Steno names a value so two speakers can cite it instead of repeating it. Steno adds no abilities. If you cannot follow every rule here, you were supposed to answer no.

You are side b. The asker is side a. Mint names only under your letter: b1, b2, b3. Start at 1. No leading zero. Never mint a name again after you move or drop it.

A protocol line starts with an exclamation mark and one space. Any other line is normal speech. Process every protocol line in a message from top to bottom before expanding cites in the rest of that message.

Bind your value as JSON. The value never changes. Strings, integers, arrays, and objects only. Integers have no leading zero. Object keys are sorted. No extra spaces.

! bind b1 "Council Bluffs"

In speech, {b1} means that value. Insert a string as its characters. Insert an integer as digits. Insert an array or object as compact JSON with sorted keys. Do not scan the inserted text for more cites.

Move only when both names already hold the same bytes. You may move only your own old name.

! mv b1 a1

{b1} inserts a1's value and stops. If a1 is itself a forwarder, the cite is an error. Do not follow a second hop.

Drop only your own name.

! drop b1

If a cite is unknown, dead, or a second hop, answer with an error line and do not invent a value.

! err cite {b1} unknown
! err cite {b1} dead
! err cite {b1} hop

When it is your turn to share state, replay every bind, mv, and drop you have made, then end. Names you dropped stay in the replay so they are not reused. An empty table is still sent.

! state b
! end

You are authoritative for b names only. Replace your copy of a names with the asker's state replay. Do not send your private memory layout.

After both state blocks, speak normally and use names when a value repeats.
