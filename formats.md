# File extensions of OTDR instruments

Most OTDRs can save a trace as a Bellcore/Telcordia `.sor` file. Many also keep their own native format, and field crews often hand over whatever the instrument saved by default. This table lists the extensions you are likely to meet and who writes them.

| Extension | Written by | Notes |
|---|---|---|
| `.sor` | Almost every OTDR | Telcordia SR-4731, versions 1.x and 2.x. See the [format map](docs/sor-format.md). |
| `.msor` | EXFO, VIAVI | Several wavelengths in one file. |
| `.trc` | EXFO, Anritsu | The same extension is used by unrelated formats from different makers. |
| `.trcx` | EXFO | Newer EXFO trace file. |
| `.bdr` | EXFO | Bidirectional result: traces from both ends of one fiber. |
| `.csor` | VIAVI | SmartOTDR. |
| `.tst` | Fluke Networks | OptiFiber and OptiFiber Pro. |
| `.tfw` | Wavetek, Acterna | Older instruments. |
| `.wtk` | Wavetek | MTS 5000. |

All of these open in the browser at [OTDR Master](https://otdrmaster.com/en/?utm_source=github&utm_medium=referral&utm_campaign=sor-format), and can be saved from there as `.sor` so that any other OTDR software reads them. If your file does not open, [send it to us](mailto:support@otdrmaster.com) and we will look at it.
