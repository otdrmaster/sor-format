# Awesome OTDR

Software, libraries and reading for people who work with OTDR traces and `.sor` files. The list is curated, not complete. Suggestions are welcome as pull requests.

## Contents

- [Format specification](#format-specification)
- [Online viewers](#online-viewers)
- [Desktop software](#desktop-software)
- [Open-source parsers](#open-source-parsers)
- [Learning](#learning)
- [Reference data](#reference-data)

## Format specification

- [Telcordia SR-4731](https://telecom-info.njdepot.ericsson.net/site-cgi/ido/docs.cgi?ID=SEARCH&DOCUMENT=SR-4731) - The standard behind `.sor` files (Optical Time Domain Reflectometer Data Format). Paid document.
- [The .sor file format, block by block](docs/sor-format.md) - A free map of the standard blocks, versions 1 and 2, in this repository.

## Online viewers

- [OTDR Master](https://otdrmaster.com/?utm_source=github&utm_medium=referral&utm_campaign=sor-format) - Opens `.sor`, `.msor`, `.trc` and other OTDR files in the browser, with event table, cursors and PDF report. Free, no install.
- [onlineotdr.com](https://onlineotdr.com/) - Online `.sor` viewer with PDF report.
- [otdrconverter.online](https://otdrconverter.online/) - Converts OTDR files to PDF and Excel reports.
- [VeEX Fiberizer Cloud](https://www.fiberizer.com/) - Cloud storage and analysis of test results from VeEX, with a free account tier.

## Desktop software

- [SORTraceViewer](https://sortraceviewer.ru/en.html) - Windows and Linux viewer and editor for `.sor` and several vendor formats.
- [EXFO FastReporter](https://www.exfo.com/en/products/field-network-testing/fiber-optic-test-software/fastreporter/) - EXFO's post-processing and reporting software for Windows.
- [VIAVI FiberTrace 2 and FiberCable 2](https://www.viavisolutions.com/en-us/products/fibertrace-2-and-fibercable-2-reporting-software) - VIAVI's trace analysis and cable reporting software for Windows.
- [AFL TRM](https://www.aflglobal.com/en/apac/products/test-and-inspection/test-management-and-reporting-software/trm-20--30---test-results-manager-pc-analysis-and-reporting) - AFL Test Results Manager for Windows.

## Open-source parsers

- [otdrs](https://github.com/JamesHarrison/otdrs) - Rust. Reads and writes `.sor`, converts to JSON.
- [pyOTDR](https://github.com/sid5432/pyOTDR) - Python. Parses `.sor` and dumps the trace.
- [jsOTDR](https://github.com/sid5432/jsOTDR) - JavaScript for Node.js, from the author of pyOTDR.
- [pubOTDR](https://github.com/sid5432/pubOTDR) - Perl, the first of the three.
- [Sor-Viewer](https://github.com/moosler/Sor-Viewer) - TypeScript. A simple viewer that runs in the browser.

## Learning

- [How to read an OTDR trace](https://otdrmaster.com/guide/how-to-read-otdr-trace?utm_source=github&utm_medium=referral&utm_campaign=sor-format) - Walks through a trace from the launch fiber to the fiber end.
- [OTDR event types](https://otdrmaster.com/guide/otdr-event-types?utm_source=github&utm_medium=referral&utm_campaign=sor-format) - Splices, connectors, bends, ghosts and what each looks like on the trace.
- [OTDR dead zone](https://otdrmaster.com/guide/otdr-dead-zone?utm_source=github&utm_medium=referral&utm_campaign=sor-format) - Event and attenuation dead zones, with a [calculator](https://otdrmaster.com/otdr-dead-zone-calculator?utm_source=github&utm_medium=referral&utm_campaign=sor-format).
- [Launch cable](https://otdrmaster.com/guide/launch-cable?utm_source=github&utm_medium=referral&utm_campaign=sor-format) - Why the first and last connector need one.
- [Bidirectional OTDR testing](https://otdrmaster.com/guide/bidirectional-otdr?utm_source=github&utm_medium=referral&utm_campaign=sor-format) - Why splice loss differs by direction and how averaging fixes it.
- [VIAVI: OTDR testing](https://www.viavisolutions.com/en-us/what-otdr-testing) - Vendor introduction to OTDR testing.
- [EXFO: OTDR](https://www.exfo.com/en/resources/glossary/otdr/) - Vendor glossary entry.
- [Fluke Networks: OTDR basics](https://www.flukenetworks.com/knowledge-base/applicationstandards-articles-books/otdr-basics) - Knowledge base article.
- [FOA: OTDRs](https://www.thefoa.org/tech/ref/testing/OTDR/OTDR.html) - The Fiber Optic Association reference page.

## Reference data

- [OTDR glossary in 15 languages](glossary/README.md) - Trace terms as technicians say them, in this repository.
- [File extensions of OTDR instruments](formats.md) - Which instruments write which files, in this repository.
