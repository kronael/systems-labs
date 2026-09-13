> Spoilers. Open only when stuck.

# Permissionless application hosting — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

Faults fire at named barriers, never on a timer and never at random. The
schedule includes:

- at the barrier where release 17's pointer update is acknowledged, the
  publishing node is stopped and stays stopped; a cold agent then asks for the
  current release;
- at a named barrier, the pins protecting release 9 on one named standing
  peer are removed and that peer collects garbage; a cold agent then fetches
  release 9 by its own address;
- the pointer update for release 21 is withheld at the boundary until a named
  barrier passes, so the pointer stays on release 20 while release 21 exists;
  agents keep resolving throughout the window;
- the pointer maintenance path is suspended and the clock is advanced past
  the pointer's own validity; agents keep resolving;
- release 12 changes the program; an agent that fetched release 11's
  interface before the change continues to drive the program after it;
- after the terminal release, the harness attempts a program upgrade with the
  same credentials the run used, and then uses the application.

Verification observes only the user-side node and the validator's RPC. It
records, for every agent and every fetch, the release identity presented and
a checksum of the bytes delivered, and it compares the board's final state
against an independently computed one. It does not inspect the publishing
path and does not require a named replication or pointer mechanism.
