# srt-to-eaf-converter

A standalone browser-based tool for converting SRT transcription files (produced by Whisper AI with speaker diarization) into EAF files (ELAN Annotation Format).

Developed for the **DOC-STL** project at Laboratoire STL (UMR 8163 CNRS), Université de Lille. Distributed as part of the [CORLI](https://corli.huma-num.fr/) consortium toolset.

---

## Overview

Automatic speech recognition pipelines such as Whisper combined with speaker diarization (e.g. via pyannote) produce SRT files where each segment is prefixed with a speaker identifier (`Speaker 0:`, `Speaker 1:`, etc.). This tool converts that output into a valid ELAN EAF file, with one tier per detected speaker, preserving all timecodes. No installation or server connection is required: the entire conversion runs locally in the browser.

---

## Features

- Drag-and-drop or file-picker interface for the SRT file
- Optional linking of the associated audio or video file (generates a relative media path in the EAF header)
- Automatic detection of speaker count and assignment to named tiers
- Conversion of all SRT timecodes to milliseconds
- Pre-conversion summary: segment count, speaker count, total duration
- Valid EAF output compatible with ELAN 3.0 and later
- UTF-8 input with BOM handling
- Bilingual interface (French / English)
- No dependencies, no external requests, no data leaves the browser

---

## Expected Input Format

The SRT file must follow the speaker-prefixed format produced by Whisper with diarization:

```
1
00:00:00,220 --> 00:00:02,773
Speaker 1: So, have you finally finished watching Dark or not?

2
00:00:02,773 --> 00:00:09,102
Speaker 0: Dark, yeah, I actually thought you knew, but for me Dark...
```

Requirements:
- Speaker labels must follow the pattern `Speaker N:` (case-insensitive, where N is an integer)
- Timecodes must follow the standard SRT format: `HH:MM:SS,mmm --> HH:MM:SS,mmm`
- Encoding: UTF-8 (with or without BOM)
- Segments without a recognized speaker prefix are currently silently skipped

---

## Output Format

The generated EAF file contains:

- One ELAN tier per detected speaker, named `Automatic_Transcription:speaker0`, `Automatic_Transcription:speaker1`, etc.
- All timecodes converted to integer milliseconds in a shared `TIME_ORDER` block
- Alignable annotations with the original text content (XML-escaped)
- An optional `MEDIA_DESCRIPTOR` element with a relative media URL (`./filename`) if a media file was provided
- A `LINGUISTIC_TYPE` declaration (`default-lt`, time-alignable) and all required ELAN constraint stubs

EAF schema: `http://www.mpi.nl/tools/elan/EAFv3.0.xsd`

Example tier structure:

```xml
<TIER LINGUISTIC_TYPE_REF="default-lt" TIER_ID="Automatic_Transcription:speaker0">
    <ANNOTATION>
        <ALIGNABLE_ANNOTATION ANNOTATION_ID="a0_1" TIME_SLOT_REF1="ts1" TIME_SLOT_REF2="ts2">
            <ANNOTATION_VALUE>Transcribed text here</ANNOTATION_VALUE>
        </ALIGNABLE_ANNOTATION>
    </ANNOTATION>
</TIER>
```

---

## Usage

1. Open `srt-to-eaf-converter.html` in any modern browser.
2. Drop your `.srt` file onto the first drop zone, or click to browse.
3. (Optional) Drop or select the corresponding audio/video file. This embeds a relative media path in the EAF header so ELAN can load the media automatically.
4. Review the detected speakers, segment count, and duration.
5. Click **Convert to EAF**. The `.eaf` file downloads immediately.
6. Place the `.eaf` file and the media file in the same folder before opening in ELAN.

---

## Typical Workflow (Huma-Num / ShareDocs)

1. Upload an audio file to ShareDocs for automatic transcription.
2. Retrieve the `.srt` output (contains timecodes and speaker labels).
3. Download the original audio file.
4. Open this converter, load the `.srt`, optionally link the audio, and convert.
5. Place the `.eaf` and the audio in the same folder.
6. Open the `.eaf` in ELAN: the media loads automatically and all speaker tiers are ready for manual revision.

---

## Limitations

- Only handles the `Speaker N:` diarization format. Other Whisper output variants (plain SRT without speaker labels, VTT, JSON) are not supported.
- Segments without a recognized speaker prefix are silently dropped. A future version should collect these into a fallback tier or report them.
- Speaker names remain generic (`speaker0`, `speaker1`, etc.); renaming must be done in ELAN after import.
- No correction of ASR transcription errors.
- No post-processing or linguistic annotation.

---

## Compatibility

- Browsers: any modern browser (Chrome, Firefox, Safari, Edge)
- Operating systems: Windows, macOS, Linux
- ELAN: version 3.0 and later

---

## Technical Notes

- Pure HTML5/CSS3/vanilla JavaScript, no build step, no external libraries
- File reading via the FileReader API; XML generation via string templating
- Time slot deduplication: shared boundary timestamps between adjacent segments map to a single `TIME_SLOT` entry
- XML special characters in annotation values are escaped (`&`, `<`, `>`, `"`)
- The output filename is derived from the input SRT filename (`.srt` extension replaced by `.eaf`)

---

## Repository Structure

```
elan-helpers/
└── srt-to-eaf-converter/
    ├── README.md
    └── srt-to-eaf-converter.html
```

---

## License

Developed within the DOC-STL research project, Laboratoire STL (UMR 8163 CNRS), Université de Lille, in partnership with the CORLI consortium (Huma-Num / CLARIN-FR).

---

## Acknowledgements

CORLI consortium -- Huma-Num research infrastructure -- CLARIN
