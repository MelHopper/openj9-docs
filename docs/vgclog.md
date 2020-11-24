## XML tags and attributes

### Garbage collection initialization configuration 

**`<initialized>`**

### *Stop-the-world* events

The following tags are used to record details about *stop-the-world* events:

**`<exclusive-start>`** and **`<exclusive-end`>**

The following table shows XML tags are nested inside:

|XML tag| nested inside | attribute| Description| policies|
|------|--------------|-----------|-------------|--------|
| `<allocation-taxation>`| `<exclusive-start>`| | balanced,  |
|`<warning`| `<exclusive-start`| `details` |balanced, |



### Concurrent events

The following tags are used to record details about concurrent GC events:

**`<concurrent-kickoff>`**

`id`

`timestamp`

|XML tag| nested inside | attribute| Description| policies|
|------|--------------|-----------|-------------|--------|
| `<kickoff>`| `<concurrent-kickoff>`|`reason` | gencon,  |
|`<warning`| `<exclusive-start`| `details` |balanced, |

**`<concurrent-collection-start>`** and **`<concurrent-collection-end>`**

**`<concurrent-start>`** and **`<concurrent-end>`**

`id`

`type `
- GMP work packet processing

`context id`

`timestamp`

**Nested tags**


|XML tag| nested inside | attribute| Description| policies|
|------|--------------|-----------|-------------|--------|
| `<concurrent-mark-start>`| `<concurrent-start>`| scanTarget| balanced,  |
|`<concurrent-mark-end>`| `<concurrent-end>`| bytesScanned, reasonForTermination|balanced, |


### GC cycles

`type`

- partial GC (balanced policy)


### GC increments

### GC operations

### Allocation failure

**`<af-start>`**

