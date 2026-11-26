# Native calls to Java

The stripped player mock looks up `demo/Player.seek(I)V` in two places. The
first native method invokes its own lookup. A second native method reads an
ID saved by a helper during `JNI_OnLoad`. Query the full managed signature
to find both native callers.

```sh
kuna xrefs tests/fixtures/android-java-calls/aarch64-linux-android.so \
  --to 'Ldemo/Player;->seek(I)V'
```

Both rows have a native call address and a separate lookup instruction. A
lookup class identifies the class used to resolve the ID. The VM chooses
the implementation for virtual dispatch at runtime. The same mock has a
conditional choice of two IDs, a global overwritten through another function,
and a primitive native entry that calls through an unrelated table. Only the
first two produce Java call records; the overwritten ID remains incomplete.

The option's speed record times single-function decompilation of the cached
caller. Each measured run loads the ELF in a fresh release console and checks
whether the call annotation appeared. From the repository root, run
`taskset -c 30 python3 -m docs.features.jniregister.speed --option jnicalls`
with an available CPU. The benchmark alternates the order for 100 pairs after
six warm-up pairs and records wall time and child CPU time.

## Release timings on the AArch64 mock

| Measurement | Off median | On median | Change | Paired 95% interval |
| --- | ---: | ---: | ---: | ---: |
| Standard wall time, 31 runs per setting | 100.06 ms | 82.10 ms | -17.95% | — |
| Alternating pairs, wall time | 65.988 ms | 68.713 ms | +4.13% | +2.96% to +4.95% |
| Alternating pairs, child user + system CPU | 64.258 ms | 66.838 ms | +4.02% | +3.05% to +4.65% |

The standard wall-time gate passed the 5% budget. The paired wall-time and child CPU changes also fell within that budget.
The standard pipeline timed every off run before the on runs. Alternating the
settings measured an additional 2.725 ms of median wall time. Both runs used CPU 30.
