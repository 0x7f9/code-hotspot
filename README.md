# code-hotspot

Named start and stop timers for code hotspots.

Create one Timer for each loop or thread. Call start with a static name. It returns a Sample. Call stop with that Sample. Stop records the wall time for that section. 

The `enabled` feature is off by default. Without that feature, start, stop, and write_report compile to empty functions.

## Install

```toml
[dependencies]
code-hotspot = { version = "0.1", default-features = false }

[features]
profile = ["code-hotspot/enabled"]
```

Build your package with the feature that turns code-hotspot on:

```bash
cargo run --features profile
```

## Start and stop

```rust
use hotspot_rs::Timer;

let mut timer = Timer::new();

let stage_a = timer.start("stage_a");
// work
timer.stop(stage_a);

let stage_b = timer.start("stage_b");
// work
timer.stop(stage_b);

timer.flush_if_due();
```

Parent and child can overlap. Timed total can exceed the window.

```rust
let pipeline = timer.start("aim.pipeline");
let vigem = timer.start("aim.vigem");
timer.stop(vigem);
timer.stop(pipeline);
```

## Time a block

```rust
let value = timer.scope("stage_c", || {
    // work
});
```

## Report

Each flush replaces the file. A new process starts with empty stats and writes to the same path.

The header shows window time, section count, sample count, timed total, call rate, and share of the window. Timed total can exceed the window when parent and child names overlap.

The table lists each name with count, total, avg, min, max, and percent of the timed total. Units sit in a fixed column. A bar shows that percent. A blank line splits name groups that use a different prefix.

```
hotspot report
window     1.71 s
sections   4
samples    341,084
timed      1.63 s
rate       199,464/s
share      95.2% of window

name                                       count       total         avg         min         max      %
-------------------------------------------------------------------------------------------------------
aim.vigem                                 85,271      1.20 s     14.0 µs      300 ns    373.5 µs   73.7  
aim.pipeline                              85,271    265.81 ms     3.1 µs      400 ns    165.8 µs   16.4  
aim.assist                                85,271     95.04 ms     1.1 µs      300 ns    404.6 µs    5.8 
aim.rcs                                   85,271     67.10 ms     786 ns      300 ns    220.3 µs    4.1 
```
