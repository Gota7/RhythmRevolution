# Wave (.brwav)
BRWAV or Binary Revolution Wave contains audio sample information.

## The Main File
The main file contains of a File Header, Info block, and a Data block. There appear to be different versions but they make no difference? The most recent version seems to be 1.2.

| **Type** | **Description** |
|----------|-----------------|
|FileHeader|File Header (Magic: RWAR)|
|Block|Table Block|
|Block|Data Block|

## Info Block
The Info block contains information about the Wave. Magic is INFO.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|SoundEncoding|Sound Encoding|
|0x01|bool|Loops|
|0x02|u8|Number of channels|
|0x03|u24|Sample rate|
|0x06|bool|Data block address is absolute if true, else relative to the Info block body (Info block start plus 8)|
|0x07|u8|Padding|
|0x08|u32|Loop start sample|
|0x0C|u32|Loop end sample|
|0x10|r32|Channel Info Table offset (0x1C)|
|0x14|u32|Data block address|
|0x18|u32|Reserved|
|0x1C|ChannelInfo[NumChannels]|Channel Info|

### Channel Info
Provides information on how to read the channel information.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|

## Data Block
The data block contains Wave information pointed to by the Table block. Each Wave is spaced by 0x20 alignment, and the first Wave starts after 0x20 alignment. Magic is DATA.