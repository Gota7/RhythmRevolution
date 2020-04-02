# Wave Sound Data (.brwsd)
BRWSD or Binary Wave Sound Data contains information about how to read Waves in a Wave Archive.

## The Main File
The main file contains of a File Header, Data block, and a Wave block.

| **Type** | **Description** |
|----------|-----------------|
|FileHeader|File Header (Magic: RSAR)|
|Block|Data Block|
|Block|Wave Block|

## File Versions
Wave Sound Datas contain 4 different versions that change how data is read and written.

| **File Version** | **Description** |
|1.0|Original version that is the most different|
|1.1|Fixes a data bug and changes the Wave block|
|1.2|Adds a send parameter|
|1.3|References Waves in Wave Archive instead of using a Wave block|

## Data Block
Contains information about how to read the Waves. Magic is DATA.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|u32|Number of Wave Sound Entries|
|0x04|Reference<`WaveSoundEntry`>[NumWaveSoundEntries]|Wave Sound Entry References|
|0x0C|WaveSoundEntry[NumWaveSoundEntries]|Wave Sound Entries|

### Wave Sound Entry
Tells how to read a Wave Sound Entry.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|Reference<`WaveSoundInfo`>|Wave Sound Info Reference|
|0x08|Reference<`Table<Reference<TrackInfo>>`>|Tracks Table Reference|
|0x10|Reference<`Table<Reference<NoteInfo>>`>|Note Table Reference|
|0x18|Table<`Reference<TrackInfo>`>|Track Table|
|----|TrackInfo[NumTracks]|Tracks|
|----|Table<`Reference<NoteInfo>`>|Note Table|
|----|NoteInfo[NumNotes]|Notes|

#### Wave Sound Info
Contains information about the Wave Sound Entry.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|f32|Pitch. Is always 1 if the version is 1.0|
|0x04|u8|Pan. 64 is middle with 127 being right and 0 being left. Is always 64 if version is 1.0|
|0x05|u8|Surround Pan. Always 0 if version is 1.0|
|0x06|u8|FX Send A|Always 0 in versions < 1.2|
|0x07|u8|FX Send B|Always 0 in versions < 1.2|
|0x08|u8|FX Send C|Always 0 in versions < 1.2|
|0x09|u8|Main Send|Always 127 in versions < 1.2|
|0x0A|u8[2]|Padding|
|0x0C|Reference<`null`>|Null Reference to Graph Envelope Table, never was implemented|
|0x14|Reference<`null`>|Null Reference to Randomizer Table, never was implemented|
|0x1C|u32|Reserved|

#### Track Info
Contains a table of Note Events for each track.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|Reference<`Table<Reference<NoteEvent>>`>|Reference to Note Event Table|
|0x08|Table<`Reference<NoteEvent>`>|Note Event Table|
|----|NoteEvent[NumNoteEvents]|Note Events|

##### Note Event
A Note Event to play on the track.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|f32|Note Position|
|0x04|f32|Note Length|
|0x08|u32|Note Index|
|0x0C|u32|Reserved|

#### Note Info
Contains information on how to play a note.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|s32|Wave Index (either in Wave block or Wave Archive)|
|0x04|u8|Attack|
|0x05|u8|Decay|
|0x06|u8|Sustain|
|0x07|u8|Release|
|0x08|u8|Hold|
|0x09|u8[3]|Padding|
|0x0C|u8|Original Key|
|0x0D|u8|Volume|
|0x0E|u8|Pan. 64 is middle with 127 being right and 0 being left. Is always 64 if version is 1.0|
|0x0F|u8|Surround Pan. Always 0 if version is 1.0|
|0x10|f32|Pitch. Is always 1 if the version is 1.0|
|0x14|Reference<`null`>|Null Reference to LFO Table, never was implemented|
|0x1C|Reference<`null`>|Null Reference to Graph Envelope Table, never was implemented|
|0x24|Reference<`null`>|Null Reference to Randomizer Table, never was implemented|
|0x2C|u32|Reserved|

## Wave Block
Contains information about how to read the Waves. Waves are most likely 0x20 aligned. Magic is WAVE. Note, this block can be null and is if the file version is 1.3 or greater.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|u32|Number of waves (only present in versions >= 1.1)|
|0x04|r32[NumWaves]|Offsets to Wave files relative to the start of the Wave block, not the Wave block body|
|----|Wave[NumWaves]|Wave files|