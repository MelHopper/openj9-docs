# `gencon` policy 

This topic covers examples of log output for the different GC cycle types that are associated with the `gencon` policy. For each type of cycle, you will learn how to interpret the XML elements in the example log and determine the characteristics of the cycle that has been recorded.

As detailed in [`gencon` Policy (Default)](gc.md#gencon-policy-default), the `gencon` policy uses two types of cycle to perform GC – a partial GC cycle and a global GC cycle. The partial GC cycle has a default and non-default mode.

The start of a `gencon` cycle is recorded in the log by the following elements and attributes. To locate a particular type of cycle, you can either search for the `“type”` attribute value of the `<cycle-start>` element or for the elements that log the cycle trigger:

| GC cycle | Value of `type` attribute| Element for cycle trigger|  Triggering reason|
|----------|----------------------------|------------------------------|------------------|
|Global    | `global`                   | `<concurrent-kickoff>`       | Threshold reached. Cycle trigger element is located after the `cycle-start>' element| 
|Partial | `scavenge`                   | `<af-start>`                 |Allocation failure. Cycle trigger element is located before the `cycle-start` element|

You can analyze the increments and operations that are associated with a particular type of cycle by locating and interpreting the elements in the following table:


| GC process             | Elements that log the start and end of the event  | Details |
|------------------------|------------------------------|------------------------------|
|GC cycle                |`<cycle-start>`, `<cycle-end>`| The start and end of a GC cycle.|
|GC STW increment        |`<gc-start>`, `<gc-end>`      | The start and end of a GC increment that begins with a pause. 
|GC STW increment        | `<concurrent-kickoff>`       | The start of the initial GC increment of the global concurrent cycle that begins the initial mark operation 
|GC STW increment        | `<concurrent-global-final> ` | The start of the final GC increment of the global concurrent cycle that executes the final collection|
|GC operations and phases| `<gc-op>`                    | A GC operation such as mark or sweep, or a suboperation phase such as class unload. |

**Note:** For more information about the XML structure of GC cycles, see [GC cycles](vgclog.md/#gc-cycles). For more information about GC cycle increments, see [GC increments and interleaving](vgclog.md#gc-increments-and-interleaving). 

The following examples use parts of a `gencon` log output to demonstrate how to analyse a log: 

- [Partial GC cycle log output (default)](#partial-gc-cycle-log-output-default)
- [Partial GC cycle log output (non-default)](#partial-gc-cycle-log-output-non-default)
- [global GC cycle log output](#global-gc-cycle-log-output)

## Partial GC cycle log output (default)
The `gencon` policy’s partial cycle runs by using a single a *stop-the-world* (STW) pause by default. The cycle consists of one STW scavenge operation, and is run on the *nursery* area only:

|GC Operation | GC increment | STW or concurrent| XML element of GC increment          | Details                                                                   |
|-------------|--------------|-------------------------------|--------------------------------------|---------------------------------------------------------------------------|
|Scavenge     |Single        | STW              | `<gc-start>`, `<gc-end>`| `<gc-op'>` |Contains detailed information about root scanning and weak roots processing|

The following example is taken from the verbose GC log output of a `gencon` policy GC. The output is broken down into sections that describe particular activities of the GC cycle. 

The output of a default partial GC cycle follows a general structure in the verbose GC log as shown. The lines are indented to help illustrate the flow and some child XML elements are omitted for clarity:

```xml
<exclusive-start> 

  <af-start> 

    <cycle-start> 

      <gc-start>  

        <mem-info> 

          <mem></mem> 

        </mem-info> 

      </gc-start> 

        <gc-op type="scavenge" > </gc-op> 

      <gc-end>  

        <mem-info> 
          
          <mem></mem> 

        </mem-info> 
      </gc-end> 

      <cycle-end> 

  <af-end> 

<exclusive-end> 
```

Let's look in detail at each of these elements.

### `<exclusive-start>`

The cycle begins with an STW pause, recorded by the <exclusive-start> XML element:  

```xml
<exclusive-start id="2" timestamp="2020-10-18T13:27:09.603" intervalms="1913.853">  

<response-info timems="0.030" idlems="0.030" threads="0" lastid="0000000000AC4600" lastname="LargeThreadPool-thread-3" />  

</exclusive-start> 
``` 

### `<af-start>`

The `<af-start>` XML element indicates that an allocation failure triggered the STW pause and subsequent GC cycle:

```xml
<af-start id="3" threadId="0000000000AC4F80" totalBytesRequested="272" timestamp="2020-10-18T13:27:09.603" intervalms="1913.921" type="nursery" />  
```

### `<cycle-start>`

The beginning of the GC cycle itself is recorded by the `<cycle-start>` XML element and contains the XML attribute `type="scavenge"` to describe the GC operation involved in this GC cycle:

```xml
<cycle-start id="4" type="scavenge" contextid="0" timestamp="2020-10-18T13:27:09.603" intervalms="1913.959" /> 
```

Logged GC events are labeled with `timestamp`,`id`, and `contextid` XML attributes. The `id` attribute increases incrementally with each GC event. All gc increments, operations, and concurrent XML elements associate with a particular cycle have a `contextid` value that matches the `id` value of the cycle.

### `<gc-start>`

The start of the first GC increment is logged by using the XML element `<gc-start>`:   

```xml 
<gc-start id="5" type="scavenge" contextid="4" timestamp="2020-10-18T13:27:09.603">  

<mem-info id="6" free="802568640" total="1073741824" percent="74">  

<mem type="nursery" free="0" total="268435456" percent="0">  

<mem type="allocate" free="0" total="134217728" percent="0" />  

<mem type="survivor" free="0" total="134217728" percent="0" />  

</mem>  

<mem type="tenure" free="802568640" total="805306368" percent="99">  

<mem type="soa" free="762302912" total="765040640" percent="99" />  

<mem type="loa" free="40265728" total="40265728" percent="100" />  

</mem>  

<remembered-set count="30758" />  

</mem-info>  

</gc-start>  

<allocation-stats totalBytes="136862352" >  

<allocated-bytes non-tlh="7097008" tlh="129765344" />  

<largest-consumer threadName="Start Level: Equinox Container: c25c3c67-836a-49e7-9949-a16d2c2a5a4d" threadId="000000000045E300" bytes="46137952" />  

</allocation-stats>   
```

The `<mem-info>` XML element and its child XML elements give detailed information about the amount of memory available and where it is located in the heap. You can determine how the GC operations in the increment modify and reclaim memory by comparing the attribute values of the `<mem-info>` and `<mem>` elements.

For this example, at the start of the GC increment the available memory is configured in the heap as follows: 

- 0% of the *nursery* area, which is split between the *allocate* and *survivor* spaces, is available as free memory.  

- 99% of the *tenure* area is available as free memory, which consists of short object allocation and long object allocation areas. 

### `<gc-op>`

The `<gc-op>` XML element and its child XML elements contain information about the increment’s operations and phases. The example GC cycle includes only 1 GC operation, which is of type `"scavenge"`:

```xml 
<gc-op id="7" type="scavenge" timems="30.675" contextid="4" timestamp="2020-10-18T13:27:09.634">  

<scavenger-info tenureage="1" tenuremask="fffe" tiltratio="50" />  

<memory-copied type="nursery" objects="461787" bytes="25522536" bytesdiscarded="367240" />  

<finalization candidates="317" enqueued="157" />  

<ownableSynchronizers candidates="6711" cleared="2579" />  

<references type="soft" candidates="9233" cleared="0" enqueued="0" dynamicThreshold="32" maxThreshold="32" />  

<references type="weak" candidates="13149" cleared="4781" enqueued="4495" />  
<references type="phantom" candidates="359" cleared="8" enqueued="8" />  

<object-monitors candidates="79" cleared="36" />  

</gc-op>  
```

The child XML elements of `<gc-op>` provide details about the operation itself. For example, the `<scavenger-info>` XML element shows that the *tenure age* is set to `"1"` for this operation. For a *tenure age* of `"1"`, any objects that are already copied between the *allocate* and *survivor* space once will be moved to the *tenure* area during the next scavenge operation. 

The `<memory-copied>` XML element indicates that 25 MB (25522536) were reclaimed in the *nursery* area for new object allocation. For more information about how the scavenge operation acts on the heap, see [`gencon` policy(default)](gc.md#gencon-policy-default).

The `contextid="7"`for the`<gc-start>` and `<gc-op>` XML elements are both ‘7’. The `id` value of the <cycle-start> element of the current partial GC cycle is also `"7"`, so the increment and operations are associated with the partial GC cycle.

### `<gc-end>`

The end of the GC increment is recorded with the `<gc-end>` XML element. The child XML elements `<mem-info>` and `<mem>` record details of the current levels of memory in different areas of the heap at the end of the GC increment, as shown in the following output:

```xml
<gc-end id="8" type="scavenge" contextid="4" durationms="30.815" usertimems="89.130" systemtimems="3.179" stalltimems="1.944" timestamp="2020-10-18T13:27:09.634" activeThreads="4">  

<mem-info id="9" free="910816000" total="1073741824" percent="84"> 

<mem type="nursery" free="108247360" total="268435456" percent="40">  

<mem type="allocate" free="108247360" total="134217728" percent="80" />  

<mem type="survivor" free="0" total="134217728" percent="0" />  

</mem>  

<mem type="tenure" free="802568640" total="805306368" percent="99">  

<mem type="soa" free="762302912" total="765040640" percent="99" />  

<mem type="loa" free="40265728" total="40265728" percent="100" />  

</mem>  

<pending-finalizers system="1" default="156" reference="4503" classloader="0" /> 

<remembered-set count="30758" /> 

</mem-info>  

</gc-end>  
```

The attribute values of `<mem>` and `<mem-info>` show that memory was released in the *nursery* area:

- 40% of the *nursery* area available as free memory. The *allocate* space of the *nursery* area is now 80% available memory, and the *survivor* space is full. 
- 99% of the *tenure* area is now available as free memory. 

### `<cycle-end>` 
The end of the GC cycle is recorded by the `<cycle-end>` XML element. The GC cycle finishes after only 1 GC increment, as shown in the following output:

```xml
<cycle-end id="10" type="scavenge" contextid="4" timestamp="2020-10-18T13:27:09.635" />  

<allocation-satisfied id="11" threadId="0000000000AC4600" bytesRequested="272" />  

<af-end id="12" timestamp="2020-10-18T13:27:09.635" threadId="0000000000AC4F80" success="true" from="nursery"/>  
 
<exclusive-end id="13" timestamp="2020-10-18T13:27:09.635" durationms="31.984" />  
```

The `<allocation-satisfied>` XML element indicates that the requested memory reclaim of 272 bytes was by the GC cycle. The `<exclusive-end>` XML element indicates the end of the STW pause.

### Summary of the example

You can now determine the following characteristics of the GC cycle from the analysis of the structure and XML elements that were covered in this example:

- The GC cycle begins with an STW pause due to an allocation failure. 

- All GC events that are associated with this cycle occur during the STW pause 

- The cycle consists of only 1 GC increment, containing only one operation, a ‘scavenge’. 

- The GC cycle reclaims memory in the *nursery* area.  
 
## Partial GC cycle log output (non-default)
 
The default [partial GC cycle ](vgclog.md#partial-gc-cycle-default) consists of a single scavenge operation, which runs start to finish during a single STW pause. The partial GC cycle can alternatively be run as a [concurrent scavenge cycle](gc.md#concurrent-scavenge) by enabling the concurrent scavenge mode. When this mode is enabled, the partial GC cycle is divided into increments to enable part of the cycle to run in parallel to running application threads. Specifically, the cycle is divided into 3 GC increments as defined in the following table:  

|GC Operation | GC increment | STW or concurrent| XML element of GC increment          | Details                                                                   |
|-------------|--------------|-------------------------------|--------------------------------------|---------------------------------------------------------------------------|
|Scavenge     |initial        | STW              | `<gc-start>`, `<gc-end>`|Root scanning, reported as a single scavenge operation |
|Scavenge     |intermediate        | concurrent         | `<concurrent-start>`, `<concurrent-end>`|`<warning details=””>`  Root scan, live objects traversed and evacuated (copy-forwarded). Reported as a scavenge operation |
|Scavenge     |final     | STW              | `<gc-start>`, `<gc-end>` |weak roots scanning, reported as a complex scavenge operation (gc-op) containing specific details for each of weak root groups  |

## Global GC cycle log output

The following example shows how a global GC cycle is recorded in a `gencon` policy verbose GC log. The global GC cycle is run after the completion of many partial GC cycles, so the log content in this example begins part way down the full log. For more information about the GC Initialization section see [Verbose GC log contents and structure ](vgclog-md#verbose-gc-log-contents-and-structure). For an example log output for a `gencon` partial cycle, see [Example - `gencon`’s default partial GC cycle](./vgclogs.md/#example-gencons-default-partial-gc-cycle). 

 [The global GC cycle is split into increments](./vgclog.md#gc-increments-and-interleaving) that interleave with partial GC cycles. The interleaving can be seen in the following example, where a partial GC cycle is logged between the start and end of the global cycle. 

 The following elements log the GC increments and operations of the global GC cycle:

 |GC Operation         | GC increment| STW or concurrent| XML element of GC increment| Details                         |
|---------------------|-------------|-------------------------------|--------------------------------------|-----------------------|
|n/a - initiates cycle|initial      | STW              | `<concurrent-kickoff`        |No `<gc-op>` is logged. This increment just initiates the concurrent mark increment |
|concurrent mark      |intermediate |concurrent                     | `<gc-start>`, `<gc-end>`     |`<concurrent-trace-info>` records progress of concurrent mark|
|final collection     |final        | STW              | `<concurrent-global-final>`  |Operations and phases include a final phase of concurrent mark, a sweep, and an optional class unload and compact. Triggered when card-cleaning threshold reached. Child XML element is `<concurrent-trace-info reason=””>`  |

The global GC cycle follows a general structure in the verbose GC log as shown. The lines are indented to help illustrate the flow and some child XML elements are omitted for clarity:

```xml
<concurrent-kickoff> <!-- the first increment of the global cycle is recorded-->

<exclusive-start></exclusive-start> <!-- STW pause starts-->

<cycle-start> <!--global cycle starts-->

<exclusive-end> <!-- STW pause ends-->

<!-- partial GC cycle events start -->

<exclusive-start> 

  <af-start> 

  <cycle-start> 

    <gc-start>  

      <mem-info> 

        <mem></mem> 

      </mem-info> 

    </gc-start> 

    <gc-op> </gc-op> 

    <gc-end>  

      <mem-info> 

        <mem></mem> 

      </mem-info> 
    </gc-end> 

    <cycle-end> 

  <af-end> 

<exclusive-end> 

<!-- partial GC cycle events end -->
 
<!--STW pause starts ready for running final increment of the global GC cycle-->

<exclusive-start> </exclusive-start> 

<concurrent-global-final></concurrent-global-final> <!--records the final increment information of global cycle-->

<gc-start> <!-- final increment begins-->

<mem-info> 

<mem></mem> 

</mem-info> 

</gc-start> 

<gc-op> “type=rs-scan"</gc-op> 

<gc-op>”type=card-cleaning" </gc-op> 

<gc-op> “type=mark”</gc-op> 

<gc-op> “type=classunload”</gc-op> 

<gc-op ”type=sweep” />

<gc-end>  <!-- final increment of the global cycle ends-->

<mem-info> 

<mem></mem> 

</mem-info> 
</gc-end> 

</cycle-end> 

<exclusive-end> <!-- final increment of the global cycle ends>

<!-- partial GC cycle events start -->
...

```

### `<concurrent-kickoff>` 

The XML element `<concurrent-kickoff>` records the start of the first increment of the `gencon` Global GC cycle. The `<kickoff>` element contains:

- Details of the reason for the launch of the GC cycle
- The target number of bytes the cycle aims to free up in the heap
- The current available memory in the different parts of the heap

```xml
<concurrent-kickoff id="12362" timestamp="2020-10-18T13:35:44.341"> 

<kickoff reason="threshold reached" targetBytes="239014924" thresholdFreeBytes="33024922" remainingFree="32933776" tenureFreeBytes="42439200" nurseryFreeBytes="32933776" /> 

</concurrent-kickoff> 
```

**Note:** To analyze specific parts of a cycle, you can search for the XML elements that mark a specific increment of the cycle. For example, you can search for the <concurrent-kickoff> XML element to locate the first increment of the `gencon` global cycle. See the details of a particular cycle, such as the [`gencon` Global Cycle](./vgclog.md/#global-gc-cycle), to determine the XML element names for particular STW or concurrent GC increments or operations. 

### `<exclusive-start>` 

The `<exclusive-start>` XML element indicates the start of an STW pause:

```xml
<exclusive-start id="12363" timestamp="2020-10-18T13:35:44.344" intervalms="342.152"> 

<response-info timems="0.135" idlems="0.068" threads="3" lastid="00000000015DE600" lastname="LargeThreadPool-thread-24" /> 

</exclusive-start> 
```

### `<cycle-start>` 

The beginning of the global cycle is recorded, indicated by the `"global"` value of the XML `type` attribute. All subsequent GC events recorded in the logs that are associated with this particular cycle have a `contextid` value equal to the `<cycle-start>` `id` attribute value of `“12634”`.

```xml
<cycle-start id="12364" type="global" contextid="0" timestamp="2020-10-18T13:35:44.344" intervalms="516655.052" /> 
```

### `<exclusive-end>` and the concurrent GC events*

The `<exclusive-end>` XML element records the end of the STW pause:

```xml
<exclusive-end id="12365" timestamp="2020-10-18T13:35:44.344" durationms="0.048" /> 
```

### Absence of event logging for 1 line
A blank line appears in the log before the next section. A blank line indicates that there no STW activities are running. However, concurrent activities might be running, in this case, the concurrent operations, and phases of the second increment of a `gencon` global cycle are running.

### Partial GC cycle starts and completes
 The next section of the logs records an STW pause that is associated with an allocation failure. The following XML element, `<cycle-start>`, indicates the start of a scavenge cycle. The ‘’contextid” XML attribute value of the XML elements in the following log section is “12368” not “12364. So the activities that are recorded in the section are associated with this new scavenge cycle rather than the currently running global cycle.  

This new scavenge cycle is a `gencon` default partial GC cycle. For more information about how this cycle is recorded in the logs, see [Example - `gencon`’s default partial GC cycle](vgclog.md#example-gencons-default-partial-gc-cycle). 

```xml
<exclusive-start id="12366" timestamp="2020-10-18T13:35:44.582" intervalms="237.874"> 

<response-info timems="0.094" idlems="0.033" threads="5" lastid="00000000014E0F00" lastname="LargeThreadPool-thread-67" /> 

</exclusive-start> 

<af-start id="12367" threadId="00000000013D7280" totalBytesRequested="96" timestamp="2020-10-18T13:35:44.582" intervalms="580.045" type="nursery" /> 

<cycle-start id="12368" type="scavenge" contextid="0" timestamp="2020-10-18T13:35:44.582" intervalms="580.047" /> 

<gc-start id="12369" type="scavenge" contextid="12368" timestamp="2020-10-18T13:35:44.582"> 

<mem-info id="12370" free="42439200" total="1073741824" percent="3"> 

<mem type="nursery" free="0" total="268435456" percent="0"> 

<mem type="allocate" free="0" total="241565696" percent="0" /> 

<mem type="survivor" free="0" total="26869760" percent="0" /> 

</mem> 

<mem type="tenure" free="42439200" total="805306368" percent="5"> 

<mem type="soa" free="2173472" total="765040640" percent="0" /> 

<mem type="loa" free="40265728" total="40265728" percent="100" /> 

</mem> 

<remembered-set count="23069" /> 

</mem-info> 

</gc-start> 

<allocation-stats totalBytes="235709920" > 

<allocated-bytes non-tlh="104" tlh="235709816" /> 

<largest-consumer threadName="LargeThreadPool-thread-79" threadId="00000000013F0C00" bytes="7369720" /> 

</allocation-stats> 

<gc-op id="12371" type="scavenge" timems="11.110" contextid="12368" timestamp="2020-10-18T13:35:44.593"> 

<scavenger-info tenureage="14" tenuremask="4000" tiltratio="89" /> 

<memory-copied type="nursery" objects="158429" bytes="6018264" bytesdiscarded="108848" /> 

<ownableSynchronizers candidates="1701" cleared="1685" /> 

<references type="soft" candidates="57" cleared="0" enqueued="0" dynamicThreshold="32" maxThreshold="32" /> 

<references type="weak" candidates="514" cleared="406" enqueued="406" /> 

<object-monitors candidates="182" cleared="0" /> 

</gc-op> 

<gc-end id="12372" type="scavenge" contextid="12368" durationms="11.249" usertimems="43.025" systemtimems="0.000" stalltimems="1.506" timestamp="2020-10-18T13:35:44.593" activeThreads="4"> 

<mem-info id="12373" free="277876128" total="1073741824" percent="25"> 

<mem type="nursery" free="235436928" total="268435456" percent="87"> 

<mem type="allocate" free="235436928" total="241565696" percent="97" /> 

<mem type="survivor" free="0" total="26869760" percent="0" /> 

</mem> 

<mem type="tenure" free="42439200" total="805306368" percent="5" macro-fragmented="0"> 

<mem type="soa" free="2173472" total="765040640" percent="0" /> 

<mem type="loa" free="40265728" total="40265728" percent="100" /> 

</mem> 

<pending-finalizers system="0" default="0" reference="406" classloader="0" /> 

<remembered-set count="17354" /> 

</mem-info> 

</gc-end> 

<cycle-end id="12374" type="scavenge" contextid="12368" timestamp="2020-10-18T13:35:44.594" /> 

<allocation-satisfied id="12375" threadId="00000000013D6900" bytesRequested="96" /> 

<af-end id="12376" timestamp="2020-10-18T13:35:44.594" threadId="00000000013D7280" success="true" from="nursery"/> 

<exclusive-end id="12377" timestamp="2020-10-18T13:35:44.594" durationms="11.816" /> 
```

### `<exclusive-start>` and `<concurrent-global-final>` 

After the partial GC cycle completes and the STW pause finishes, the log records a new STW pause, which precedes a `<concurrent-global-final>` XML element. 

The `<concurrent-global-final>` XML element records the start of the third and final increment of the Global partial GC cycle, which consists of STW GC operations and phases. The STW pause is run by the garbage collector to run this final increment.  

The `reason` attribute of the `<concurrent-trace-info>` XML element indicates that the global cycle reached the card cleaning threshold and so can now complete this final increment.  

```xml
<exclusive-start id="12378" timestamp="2020-10-18T13:35:44.594" intervalms="12.075"> 

<response-info timems="0.108" idlems="0.040" threads="3" lastid="00000000018D3800" lastname="LargeThreadPool-thread-33" /> 

</exclusive-start> 

<concurrent-global-final id="12379" timestamp="2020-10-18T13:35:44.594" intervalms="516905.029" > 

<concurrent-trace-info reason="card cleaning threshold reached" tracedByMutators="200087048" tracedByHelpers="12164180" cardsCleaned="4966" workStackOverflowCount="0" /> 

</concurrent-global-final> 
```

### `<gc-start>`

A global GC increment begins. You can check that the increment is associated with the GC global cycle in the example by checking the `contextid` XML attribute value matches the `id` XML attribute value of the cycle's <gc-cycle> XML element. For the increment in the example, the` <gc-start>` `contextid` and `<gc-cycle>` `id` value are both `"12364"`.

```xml
<gc-start id="12380" type="global" contextid="12364" timestamp="2020-10-18T13:35:44.594"> 

<mem-info id="12381" free="277048640" total="1073741824" percent="25"> 

<mem type="nursery" free="234609440" total="268435456" percent="87"> 

<mem type="allocate" free="234609440" total="241565696" percent="97" /> 

<mem type="survivor" free="0" total="26869760" percent="0" /> 

</mem> 

<mem type="tenure" free="42439200" total="805306368" percent="5"> 

<mem type="soa" free="2173472" total="765040640" percent="0" /> 

<mem type="loa" free="40265728" total="40265728" percent="100" /> 

</mem> 

<pending-finalizers system="0" default="0" reference="405" classloader="0" /> 

<remembered-set count="17388" /> 

</mem-info> 

</gc-start> 

<allocation-stats totalBytes="827488" > 

<allocated-bytes non-tlh="96" tlh="827392" /> 

<largest-consumer threadName="LargeThreadPool-thread-68" threadId="00000000013D6900" bytes="65632" /> 

</allocation-stats> 
```

The child XML element attribute values of the`<mem>` and `<mem-info>` XML elements indicate the configuration of the memory. For this example, at the start of this GC increment, 25% of the total heap is available as free memory. This free memory is split between the following areas of the heap:  

The *nursery* area, which has 87% of its total memory available as free memory. The free memory is only available in the *allocate* space of the *nursery* area. The *survivor* space has no free memory.  

The *tenure* area, which has 5% of its total memory available as free memory. All of this free memory is in the long object allocation area. No free memory is available in the short object allocation area.  

**Note:** The global GC cycle runs to free up memory in the *tenure* area. The freeing up of memory in the *nursery* area is achieved by using the partial GC cycle. For more information, see [`gencon` Policy(default)](gc.md#gencon-policy-(default)).

### `<gc-op>` 

The `<gc-op>` XML element and its child XML elements contain information about the increment’s operations and phases. 5 `<gc-op>` XML elements are recorded. The `type` XML attribute records the different operations and phases that are involved:

1. `Rs-scan`
2. `Card-cleaning`
3. `Mark`
4. `Classunload`
5. `Sweep`

**Note:** The final increment of a `gencon` global cycle can include an optional compact phase.

```xml
<gc-op id="12382" type="rs-scan" timems="3.525" contextid="12364" timestamp="2020-10-18T13:35:44.598"> 

<scan objectsFound="11895" bytesTraced="5537600" workStackOverflowCount="0" /> 

</gc-op> 

<gc-op id="12383" type="card-cleaning" timems="2.910" contextid="12364" timestamp="2020-10-18T13:35:44.601"> 

<card-cleaning cardsCleaned="3603" bytesTraced="5808348" workStackOverflowCount="0" /> 

</gc-op> 

<gc-op id="12384" type="mark" timems="6.495" contextid="12364" timestamp="2020-10-18T13:35:44.607"> 

<trace-info objectcount="1936" scancount="1698" scanbytes="61200" /> 

<finalization candidates="389" enqueued="1" /> 

<ownableSynchronizers candidates="5076" cleared="523" /> 

<references type="soft" candidates="18420" cleared="0" enqueued="0" dynamicThreshold="32" maxThreshold="32" /> 

<references type="weak" candidates="19920" cleared="114" enqueued="60" /> 

<references type="phantom" candidates="671" cleared="50" enqueued="50" /> 

<stringconstants candidates="40956" cleared="109" /> 

<object-monitors candidates="182" cleared="51" /> 

</gc-op> 

<gc-op id="12385" type="classunload" timems="1.607" contextid="12364" timestamp="2020-10-18T13:35:44.609"> 

<classunload-info classloadercandidates="425" classloadersunloaded="6" classesunloaded="2" anonymousclassesunloaded="1" quiescems="0.000" setupms="1.581" scanms="0.019" postms="0.007" /> 

</gc-op> 

<gc-op id="12386" type="sweep" timems="9.464" contextid="12364" timestamp="2020-10-18T13:35:44.618" /> 
```

### `<gc-end>`

The `<gc-end>` XML element and its child XML elements record details about the end of the final increment of the global cycle, as shown in the following output:

```xml
<gc-end id="12387" type="global" contextid="12364" durationms="24.220" usertimems="86.465" systemtimems="0.000" stalltimems="2.846" timestamp="2020-10-18T13:35:44.618" activeThreads="4"> 

<mem-info id="12388" free="650476504" total="1073741824" percent="60"> 

<mem type="nursery" free="235516088" total="268435456" percent="87"> 

<mem type="allocate" free="235516088" total="241565696" percent="97" /> 

<mem type="survivor" free="0" total="26869760" percent="0" /> 

</mem> 

<mem type="tenure" free="414960416" total="805306368" percent="51" micro-fragmented="98245682" macro-fragmented="0"> 

<mem type="soa" free="374694688" total="765040640" percent="48" /> 

<mem type="loa" free="40265728" total="40265728" percent="100" /> 

</mem> 

<pending-finalizers system="1" default="0" reference="515" classloader="0" /> 

<remembered-set count="13554" /> 

</mem-info> 

</gc-end> 
```

The `<mem>` and `<mem-info>` child XML elements show that after the increment runs, the heap contains 60% free memory. This free memory is split between the following areas of the heap:  

- the *nursery* area remains unchanged, with 87% of its total memory available as free memory. The free memory is only available in the *allocate* space of the *nursery* area. The *survivor* space has no free memory. 
- the *tenure* area, now has 51% of its total memory available as free memory. The memory is split between the small object allocation space, which has 48% of its space available as free memory, and the large object allocation space, which is all available memory.

### `<cycle-end>`

After the GC operations and phases of the final increment of the global cycle complete, the global cycle ends and the STW pause ends, as shown in the following output:

```xml
<cycle-end id="12389" type="global" contextid="12364" timestamp="2020-10-18T13:35:44.619" /> 

<exclusive-end id="12391" timestamp="2020-10-18T13:35:44.619" durationms="24.679" /> 
```

### Summary of the example

You can now determine the following characteristics of the GC cycle from the analysis of the structure and XML elements that were covered in this example:

- The GC global cycle is triggered when a memory threshold is reached and begins with an STW pause 

- After the first increment of the GC global cycle completes, the STW pause ends and the second increment runs concurrently. 

- While the second increment is running concurrently, a partial GC cycle starts and finishes. 

- When the second increment completes, an STW pause begins so that the third and final increment of the global cycle, which consists of five operations and phases, can run. 

- The global GC cycle reclaims memory in the *tenure* area.