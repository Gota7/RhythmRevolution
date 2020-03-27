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
The info block contains information about entries.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|Reference|Reference to Sound Table|
|0x08|Reference|Reference to Bank Table|
|0x10|Reference|Reference to Player Table|
|0x18|Reference|Reference to File Table|
|0x20|Reference|Reference to Group Table|
|0x28|Reference|Reference to Sound Archive Information|
|0x30|Table<`Reference`>|Sound Table|
|----|Table<`Reference`>|Bank Table|
|----|Table<`Reference`>|Player Table|
|----|Table<`Reference`>|File Table|
|----|Table<`Reference`>|Group Table|
|----|SoundInfo[NumSounds]|Sounds|
|----|BankInfo[NumBanks]|Banks|
|----|PlayerInfo[NumPlayers]|Players|
|----|FileInfo[NumFiles]|Files|
|----|GroupInfo[NumGroups]|Groups|
|----|SoundArchiveInformation|Sound Archive Information|

### Sound Info
TODO

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
TODO

### Group Info
TODO

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