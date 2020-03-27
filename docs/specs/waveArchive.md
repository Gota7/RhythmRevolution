# Wave Archive (.brwar)
BRWAR or Binary Revolution Wave Archive contains Waves.

## The Main File
The main file contains of a File Header, Table block, and a Data block.

| **Type** | **Description** |
|----------|-----------------|
|FileHeader|File Header (Magic: RWAR, Version: 1.0)|
|Block|Table Block|
|Block|Data Block|

## Table Block
The Table block contains offsets to each Wave. Magic is TABL.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|Table<`TableItem`>|Wave Items|

### Table Item
An item that points to a Wave in the Data block. All offsets are relative to the start of the Data block, not the Data block body.

| **Offset** | **Type** | **Description** |
|------------|----------|-----------------|
|0x00|Reference<`Wave`>|Reference to the Wave|
|0x08|u32|Wave size|

## Data Block
The data block contains Wave information pointed to by the Table block. Each Wave is spaced by 0x20 alignment, and the first Wave starts after 0x20 alignment. Magic is DATA.