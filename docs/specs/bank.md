# Bank (.brbnk)
BRBNK or Binary Revolution Bank contains instrument data for Sequences.

## The Main File
The main file contains of a File Header, Data block, and an optional Wave block.

| **Type** | **Description** |
|----------|-----------------|
|FileHeader|File Header (Magic: RBNK)|
|Block|Data Block|
|Block|Wave Block (Optional)|

## File Versions
Sound Archives contain 3 different versions that change how data is read and written. The version changes only effect Sound info, and nothing else.

| **File Version** | **Description** |
|1.0|Original version that is the most different.|
|1.1|Adds instrument volume and tune|
|1.2|Uses a Wave Archive for storing Waves instead of a Wave block|

## Data Block
Contains instrument data. Magic is DATA. An invalid instrument is null and has no playback information. Think of it as a placeholder.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|Table<`Reference<Instrument>(Identifer: 0 = Invalid, 1 = Direct, 2 = Range, 3 = Index)`>|Instrument Table. A reference can point to 1 of 4 different type of instrument entries|
|----|Instrument[NumInstruments]|Instruments|

### Direct Instrument
Most straightforward instrument as it has no key or velocity regions, just direct data that covers the whole keyboard.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|NotePlaybackInfo|Note Playback Info|

### Range Instrument
Splits the instrument into ranges of key regions.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|u8|Number of ranges|
|0x01|u8[NumRanges]|Ranges for each instrument. For example, the values 21, 43, and 127 represent 3 key regions: 0 - 21, 22 - 43, and 44 - 127|
|----|--|Padding to 0x4 alignment|
|----|Reference<`KeyRegion`>(Identifer: 0 = Invalid, 1 = Direct, 2 = Range, 3 = Index)[NumRanges]|Reference to each key region, which could be 1 of 4 different types|
|----|KeyRegion[NumRanges]|Key Regions|

### Index Instrument
Defines instrument data for every key in a specified range. This is useful for drumsets, or when you don't want sounds for certain lower keys.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|u8|Minimum key defined|
|0x01|u8|Maximum key defined|
|0x02|u16|Padding|
|0x04|Reference<`KeyRegion`>(Identifer: 0 = Invalid, 1 = Direct, 2 = Range, 3 = Index)[Max - Min + 1]|Data for each key region for each individual note|
|----|KeyRegion[Max - Min + `]|Key Regions|

#### Direct Key Region
Most straightforward key region as it just has data that covers for the keyboard's entire velocity.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|NotePlaybackInfo|Note Playback Info|

#### Range Key Region
Splits the key region into ranges of velocity regions.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|u8|Number of ranges|
|0x01|u8[NumRanges]|Ranges for each key region. For example, the values 21, 43, and 127 represent 3 velocity regions: 0 - 21, 22 - 43, and 44 - 127|
|----|--|Padding to 0x4 alignment|
|----|Reference<`NotePlaybackInfo`>(Identifer: 0 = Invalid, 1 = Valid)[NumRanges]|Reference to each velocity region, which could be 1 of 2 different types|
|----|NotePlaybackInfo[NumRanges]|Velocity Regions|

#### Index Key Region
Defines instrument data for every velocity in a specified range. This is useful for when you don't want sounds for certain lower velocities.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|u8|Minimum velocity defined|
|0x01|u8|Maximum velocity defined|
|0x02|u16|Padding|
|0x04|Reference<`NotePlaybackInfo`>(Identifer: 0 = Invalid, 1 = Valid)[Max - Min + 1]|Data for each velocity region|
|----|NotePlaybackInfo[Max - Min + `]|Velocity Regions|

##### Note Playback Info
Contains information on how to play a note.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|WaveReferenceUnion|Union of how to reference the Wave used by this info|
|0x04|u8|Attack|
|0x05|u8|Decay|
|0x06|u8|Sustain|
|0x07|u8|Release|
|0x08|u8|Hold|
|0x09|WaveReferenceUnionType|What type of referenced is used in the union above|
|0x0A|bool|Percussion mode. If true, ignore the release part of a note so the note will play until completion, even for looping Waves|
|0x0B|u8|Key group. If not 0, any notes playing with the same key group will be stopped|
|0x0C|u8|Root key|
|0x0D|u8|Volume. 127 if version < 1.1|
|0x0E|u16|Padding|
|0x10|f32|Tune. 1 if version < 1.1|
|0x14|Reference<`null`>|Null Reference to LFO Table, never was implemented|
|0x1C|Reference<`null`>|Null Reference to Graph Envelope Table, never was implemented|
|0x24|Reference<`null`>|Null Reference to Randomizer Table, never was implemented|
|0x2C|u32|Reserved|

###### Wave Reference Union Type
Is one byte and size and specifies how the Wave is referenced.

| **Identifier** | **Description** |
|----------------|-----------------|
|0|Wave Index|
|1|Address|
|2|Callback|

###### Wave Reference Union
Varies depending on the type. If you find a situation where the type is not Wave Index, please tell me immediately so I can update this specification.

| **Type** | **Data Type** | **Description** |
|----------|---------------|-----------------|
|Wave Index|s32|Index in the Wave Archive or Wave Block|
|Address|u32|Address to Wave file. What it's relative to is unknown|
|Callback|u32?|????????????????|

## Wave Block
Contains information about how to read the Waves. Waves are most likely 0x20 aligned. Magic is WAVE. Note, this block can be null and is if the file version is 1.2 or greater.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|Table<`Reference<WaveInfoBlockBody>`>|Reference table to wave info|
|----|Wave[NumWaves]|Wave files|