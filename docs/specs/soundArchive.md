# Sound Archive (.brsar)
BRSAR or Binary Revolution Sound Archive contains sound data for the game.

## The Main File
The main file contains of a File Header, Symbol block, an Info block, and a File block.

| **Type** | **Description** |
|----------|-----------------|
|FileHeader|File Header (Magic: RSAR)|
|Block|Symbol Block|
|Block|Info Block|
|Block|File Block|

## File Versions
Sound Archives contain 5 different versions that change how data is read and written. The version changes only effect Sound info, and nothing else.

| **File Version** | **Description** |
|1.0|Original version that is the most different. TODO: DESCRIBE HOW IT IMPACTS HOW SOUNDS ARE READ|
|1.1|Introduces the concept of having each Sound reference link to some common info, and then have that common info link to more detailed information for each Sound type|
|1.2|Introduces the use of Pan Mode and Pan Curve. Before this version the bytes were padding? For earlier versions, the default Pan Mode is Balance, and the Pan Curve is Square Root|
|1.3|TODO|
|1.4|Converts the Stream's allocated channel bitflag into a simple channel count, as the bitflag was just converted to a count anyway (as channels are sequential)|

## Symbol Block
Contains names for entries in the Info block. Magic is SYMB.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|r32|Relative offset to String Table|
|0x04|r32|Relative offset to Sound Patricia Tree|
|0x08|r32|Relative offset to Player Patricia Tree|
|0x0C|r32|Relative offset to Group Patricia Tree|
|0x10|r32|Relative offset to Bank Patricia Tree|
|0x14|Table<`r32`>|String table pointing to item names|
|----|string[StringTableCount]|Null terminated item name strings|
|----|---|Padding to 0x4 alignment|
|----|PatriciaTree|Sound Patricia Tree|
|----|PatriciaTree|Player Patricia Tree|
|----|PatriciaTree|Group Patricia Tree|
|----|PatriciaTree|Bank Patricia Tree|

### Patricia Tree
A Patricia Tree is a very efficient way of searching strings.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|u32|Root Node Index|
|0x04|u32|Number of nodes|
|0x08|PatriciaTreeNode[NumNodes]|Patricia Tree Nodes|

#### Patricia Tree Node
A node of the Patricia Tree.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|u16|1 if this is a leaf node, otherwise 0|
|0x02|u16|Bit index of the string to compare|
|0x04|u32|Node index to go to if the compared bit is 0, 0xFFFFFFFF if null|
|0x08|u32|Node index to go to if the compared bit is 1, 0xFFFFFFFF if null|
|0x0C|u32|Index in the string table for the item, 0xFFFFFFFF if null|
|0x10|u32|Item index in the INFO block|

#### Searching The Patricia Tree
When searching the bits of a string, continue reading the tree until the end of the string or a leaf node is reached. Here is some example code to do so:

```cs
//Start at the root node.
PatriciaTreeNode currNode = patriciaTreeNodes[rootNodeId];

//Continue until a leaf flag.
while ((currNode.LeafFlag & 0b1) > 0) {
	
	//Get the char position and bit index for the char.
	int pos = currNode.Bit >> 3;
	int bit = currNode.Bit & 0b111;

	//Jump to the right.
	if (pos < stringToSearch.Length && (stringToSearch[pos] & (1 << (7 - bit))) > 0) {
		currNode = patriciaTreeNodes[currNode.RightNodeId];
	}

	//Jump to the left.
	else {
		currNode = patriciaTreeNodes[currNode.LeftNodeId];
	}

}

//Return the current node, which contains information such as the string index and the item's index in the INFO block.
return currNode;
```

## Info Block
The info block contains information about entries. Magic is INFO.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|Reference<`Table<Reference<SoundInfo>>`>|Reference to Sound Table|
|0x08|Reference<`Table<Reference<BankInfo>>`>|Reference to Bank Table|
|0x10|Reference<`Table<Reference<PlayerInfo>>`>|Reference to Player Table|
|0x18|Reference<`Table<Reference<FileInfo>>`>|Reference to File Table|
|0x20|Reference<`Table<Reference<GroupInfo>>`>|Reference to Group Table|
|0x28|Reference<`Table<Reference<SoundArchiveInformation>>`>|Reference to Sound Archive Information|
|----|SoundArchiveInformation|Sound Archive Information|

### Sound Info
TODO - AND VERSION DIFFERENCES

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|IdType (Type is ???)|String Id|
|0x04|IdType (Type is ???)|File Id|
|0x08|IdType (Type is ???)|Player Id|
|0x0C|Reference<`Sound3DInfo`>|3D Sound Info|
|0x14|u8|Volume|
|0x15|u8|Player Priority|
|0x16|SoundType|Sound Type|
|0x17|u8|Remote Filter|
|0x18|Reference<`DetailedSoundInfo (Data Type is Sound Type)`>|Reference to Detailed Sound Info (Sequence, Stream, or Wave)|
|0x20|u32[2]|User Parameters|
|0x28|PanMode|Pan Mode|
|0x29|PanCurve|Pan Curve|
|0x2A|u8|Actor Player Id|
|0x2B|u8|Reserved|

#### Sound Type
A Sound Type is one byte in size and specifies the type of Sound entry.

| **Identifier** | **Description** |
|----------------|-----------------|
|0|Invalid|
|1|Sequence|
|2|Stream|
|3|Wave|

#### Pan Mode

#### Pan Curve

#### Sequence Detailed Sound Info

#### Stream Detailed Sound Info

#### Wave Detailed Sound Info

### Bank Info
Contains information about banks.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|IdType (Type is ???)|String Id|
|0x04|IdType (Type is ???)|File Id|
|0x08|u32|Reserved|

### Player Info
Contains information about players.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|IdType (Type is ???)|String Id|
|0x04|u8|Max number of sounds|
|0x05|u8[3]|Padding|
|0x08|u32|Space to reserve in the heap|
|0x0C|u32|Reserved|

### File Info
File info contains information of how to locate the file.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|u32|File size|
|0x04|u32|File size of the associated Wave Archive (just the file or subsection???)|
|0x08|s32|File entry number??? File Id???|
|0x0C|Reference<`string`>|External file name|
|0x14|Reference<`Table<Reference<FilePosition>>`>|Reference to File Position Table|

#### File Position
A File Position tells which Group to find the file in, as every file is contained within a Group (TODO: CHECK IF THIS APPLIES FOR STREAMS). The Group is actually what contains the file offsets.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|IdType (Type is ???)|Group Id|
|0x04|u32|Index in the Group|

### Group Info
Groups contain information about which files should be loaded at the same time for efficiency.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|IdType (Type is ???)|String Id|
|0x04|s32|Entry number??? Group Id???|
|0x08|Reference<`string`>|External file path|
|0x10|a32|Base absolute offset for the start of the files contained within the Group|
|0x14|u32|Total size of all files contained within the Group|
|0x18|a32|Absolute offset of the Wave Archive for the Group|
|0x1C|u32|Size of the Wave Archive for the Group|
|0x20|Reference<`Table<GroupEntry>`>|Reference to Group Entry Table|

#### Group Entry
A Group Entry represents a file contained within a Group.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|IdType (Type is ???)|File Id|
|0x04|u32|File offset relative to the Base Offset for the Group's file data|
|0x08|u32|File size|
|0x0C|u32|Wave Archive Offset relative to the Base Offset for the Group's Wave Archive Data (just the file or subsection???)|
|0x10|u32|Wave Archive data size (just the file or subsection???)|
|0x14|u32|Reserved|

### Sound Archive Information
Contains information about the Sound Archive.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|u16|Max number of Sequences|
|0x02|u16|Max number of Sequence tracks|
|0x04|u16|Max number of Streams|
|0x06|u16|Max number of Stream tracks|
|0x08|u16|Max number of Stream channels|
|0x0A|u16|Max number of Waves|
|0x0C|u16|Max number of Wave tracks|
|0x0E|u16|Padding|
|0x10|u32|Reserved|

## File Block
The file block contains file information pointed to by the Info block. Each file is spaced by 0x20 alignment, and the first file starts after 0x20 alignment. Magic is FILE.