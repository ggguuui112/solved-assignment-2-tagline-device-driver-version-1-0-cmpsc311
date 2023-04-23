Download Link: https://assignmentchef.com/product/solved-assignment-2-tagline-device-driver-version-1-0-cmpsc311
<br>
5/5 - (2 votes)

In this assignment you will write a device driver for the <code>tagline</code> device, as described in the specification here. At the highest level, you will translate tagline operations into low-level disk operations. This will require you to figure out how to store data on the disk in a way that allows later reads to be successfully completed. Read the instructions carefully and follow the instructions to create and tests your implementation. Note these instructions should be read in the context of the class presentation and slides provided <a href="http://www.cse.psu.edu/~pdm12/cmpsc311-f15/slides/cmpsc311-tagline.pdf">here</a>.

<h2>System Overview</h2>

<img decoding="async" align="right" data-recalc-dims="1" data-src="https://i0.wp.com/www.cse.psu.edu/~pdm12/cmpsc311-f15/pics/tagline-overview.jpeg?w=980" class="assignment lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" class="assignment" src="https://i0.wp.com/www.cse.psu.edu/~pdm12/cmpsc311-f15/pics/tagline-overview.jpeg?w=980" align="right" data-recalc-dims="1">

 </noscript>You are to write a basic device driver that will sit between a virtual application and virtualized hardware. The application makes use of the abstraction you will provide called tagline storage. The design of the system is shown in the figure to the right:

Described in detail below, we can see three layers. The tagline application is provided to you and will call the tagline device driver functions with “tagline operations”. these operations create and read/write data to special data abstractions called taglines. You are to write the device driver code to implement this abstraction. Your code will create taglines by sending “RAID operations” to a virtual RAID disk array using a virtual I/O bus.

All of the code for tagline application and virtual RAID array is given to you. Numerous utility functions are also provided to help debug and test your program, as well as simply the creation of readable output. Sample workloads have been generated and will be used to extensively test the program. Students that make use of all of these tools (which take a bit of time up front to figure out) will find that the later assignments will become much easier to complete.

<h3>Tagline Abstraction</h3>

A <b>tagline</b> is storage abstraction somewhat similar to a data file, with several key differences. The basic idea is that the application will create, read, and write to a tagline. Almost all of the important details of the tagline interface are defined in the <tt>tagline_driver.h</tt> header file.

<ol>

 <li>The “name” of a tagline is a 16-bit unsigned integer. Note that this is assigned by the application and managed by your device driver.</li>

 <li>The tagline consists of a zero-indexed set of <b>blocks</b> that are read from and written to the storage. A file will initially have zero blocks. All reads and writes will be performed on one or more blocks.</li>

 <li>The size of tagline blocks is defined as <tt>TAGLINE_BLOCK_SIZE</tt> in the <tt>tagline_driver.h</tt> file.</li>

 <li>taglines are monotonically increasing, meaning that they are never deleted and they never shrink. Blocks <i>can be</i> overwritten however.</li>

</ol>

For this assignment, you are to implement 4 functions in <tt>tagline_driver.c</tt>:

<pre>int tagline_driver_init(uint32_t maxlines);	// Initialize the driver with a number of maximum lines to process</pre>

This function will initialize and disk interfaces, create local data structures, or any other startup bookkeeping it needs to do to start accepting reads and writes. You will need to format each of the disks before you can use them, so you need to do that in this function.

<pre>int tagline_read(TagLineNumber tag, TagLineBlockNumber bnum, uint8_t blks, char *buf);	// Read a number of blocks from the tagline driver</pre>

This function reads <tt>blks</tt> blocks from tagline <tt>tag</tt> starting from block number <tt>bnum</tt> and place the contents into the memory region <tt>buf</tt>. Note that any attempt to read beyond the end of the tagline should result in an error (and nothing copied into the buffer).

<pre>int tagline_write(TagLineNumber tag, TagLineBlockNumber bnum, uint8_t blks, char *buf);	// Write a number of blocks from the tagline driver</pre>

This function writes <tt>blks</tt> blocks from memory region <tt>buf</tt> to tagline <tt>tag</tt> starting from block number <tt>bnum</tt>. Writes beyond the end of the tagline should increase the size of the tagline. Note that any attempt to write starting beyond the end of the tagline should result in an error (and nothing written to the tagline).

<pre>int tagline_close(void);	// Close the tagline interface</pre>

This function closes the taglines and frees up any memory and closes the RAID device interface.

Note that for this assignment <span style="color: red;">we make two restrictions that vastly reduce the complexity of the implementation</span>. First the device driver will only receive requests for a single tagline (you only have to worry about one tagline). Thus,a any request for a tagline other than the first one seen can be rejected by your code with an error. Second, all tagline reads and writes will be for a single block. Thus, you can return an error when any request asks for more or less than a single block.

<h3>RAID Abstraction</h3>

The raid array is a collection of disks each with a set of fixed sized blocks that are written and read. You are to build a software layer that implements the tagline interface on top of the raid array system (e.g., translate the tagline operations into disk operations).

<ol>

 <li>Each block is of size RAID_BLOCK_SIZE (defined in raid_bus.h).</li>

 <li>For this assignment, you can assume that there are RAID_DISKS disks, each with RAID_DISKBLOCKS blocks.</li>

 <li>The blocks are indexed 0 to RAID_DISKBLOCK-1</li>

</ol>

The RAID array is controlled by sending commands through the bus interface. You can send commands on the bus using the following function:

<code>RAIDOpCode raid_bus_request(RAIDOpCode request, void *buf);</code>Where the fields of the 64-bit opcodes are defined as follows:

<pre>Bits    Description-----   -------------------------------------------  0-7   request type 8-15   number of blocks16-23   disk number24-30   unused (for now, always set to zero)   31 - status this is the result bit (0 success)32-63   block ID</pre>

The following commands are available to manipulate the RAID array:




<table id="infotable" width="95%">

 <tbody>

  <tr>

   <td><b>Field</b></td>

   <td><b>Request Value</b></td>

   <td><b>Response Value</b></td>

  </tr>

  <tr>

   <td class="head" colspan="3"><b>RAID_INIT – Initialize the RAID interface</b></td>

  </tr>

  <tr>

   <td>Type</td>

   <td>RAID_INIT</td>

   <td>RAID_INIT</td>

  </tr>

  <tr>

   <td># Blocks</td>

   <td>this is the number of tracks to create. The disk will have this times RAID_TRACK_BLOCKS addressable blocks when done.</td>

   <td>tracks created</td>

  </tr>

  <tr>

   <td>Disk Number</td>

   <td>The number of disks to created</td>

   <td>disks created</td>

  </tr>

  <tr>

   <td>Status</td>

   <td>0</td>

   <td>0 if successful, 1 if failure (see log for details)</td>

  </tr>

  <tr>

   <td>Block ID</td>

   <td>0</td>

   <td>0</td>

  </tr>

  <tr>

   <td>buf</td>

   <td>NULL</td>

   <td class="ignore">N/A</td>

  </tr>

  <tr>

   <td class="head" colspan="3"><b>RAID_FORMAT – Format a disk in the array. This will zero the disk contents and prepare it for use.</b></td>

  </tr>

  <tr>

   <td>Type</td>

   <td>RAID_FORMAT</td>

   <td>RAID_FORMAT</td>

  </tr>

  <tr>

   <td># Blocks</td>

   <td>0</td>

   <td>0</td>

  </tr>

  <tr>

   <td>Disk Number</td>

   <td>The number of the disk to format</td>

   <td>The disk formatted</td>

  </tr>

  <tr>

   <td>Status</td>

   <td>0</td>

   <td>0 if successful, 1 if failure (see log for details)</td>

  </tr>

  <tr>

   <td>Block ID</td>

   <td>0</td>

   <td>0</td>

  </tr>

  <tr>

   <td>buf</td>

   <td>NULL</td>

   <td class="ignore">N/A</td>

  </tr>

  <tr>

   <td class="head" colspan="3"><b>RAID_READ – Read consecutive blocks in the disk array</b></td>

  </tr>

  <tr>

   <td>Type</td>

   <td>RAID_READ</td>

   <td>RAID_READ</td>

  </tr>

  <tr>

   <td># Blocks</td>

   <td>The number of blocks to write to</td>

   <td>Blocks written</td>

  </tr>

  <tr>

   <td>Disk Number</td>

   <td>The disk to read from</td>

   <td>The disk read from</td>

  </tr>

  <tr>

   <td>Status</td>

   <td>0</td>

   <td>0 if successful, 1 if failure (see log for details)</td>

  </tr>

  <tr>

   <td>Block ID</td>

   <td>The starting block number to read from</td>

   <td>The starting block number read from</td>

  </tr>

  <tr>

   <td>buf</td>

   <td>Pointer to buffer containing blocks to read</td>

   <td>Pointer to the buffer passed in</td>

  </tr>

  <tr>

   <td class="head" colspan="3"><b>RAID_WRITE – Write consecutive blocks to the disk array</b></td>

  </tr>

  <tr>

   <td>Type</td>

   <td>RAID_WRITE</td>

   <td>RAID_WRITE</td>

  </tr>

  <tr>

   <td># Blocks</td>

   <td>The number of disk blocks to write</td>

   <td>Blocks written</td>

  </tr>

  <tr>

   <td>Disk Number</td>

   <td>The disk to write to</td>

   <td>The disk written to</td>

  </tr>

  <tr>

   <td>Status</td>

   <td>0</td>

   <td>0 if successful, 1 if failure (see log for details)</td>

  </tr>

  <tr>

   <td>Block ID</td>

   <td>The starting disk block number to write to</td>

   <td>The starting disk block number written to</td>

  </tr>

  <tr>

   <td>buf</td>

   <td>Pointer to buffer containing blocks to write from</td>

   <td>Pointer to the buffer passed in</td>

  </tr>

  <tr>

   <td class="head" colspan="3"><b>RAID_CLOSE – shuts down the RAID interface</b></td>

  </tr>

  <tr>

   <td>Type</td>

   <td>RAID_CLOSE</td>

   <td>RAID_CLOSE</td>

  </tr>

  <tr>

   <td># Blocks</td>

   <td>0</td>

   <td>0</td>

  </tr>

  <tr>

   <td>Disk Number</td>

   <td>0</td>

   <td>0</td>

  </tr>

  <tr>

   <td>Status</td>

   <td>0</td>

   <td>0 if successful, 1 if failure (see log for details)</td>

  </tr>

  <tr>

   <td>Block ID</td>

   <td>0</td>

   <td>0</td>

  </tr>

  <tr>

   <td>buf</td>

   <td>NULL</td>

   <td class="ignore">N/A</td>

  </tr>

 </tbody>

</table>




<h3>Summary</h3>

To summarize, your code will receive commands from the vritual application. It will do the following functions to implement the device driver.

<ol>

 <li>Initialize the disk array by calling the init &amp; format functions</li>

 <li>On writes for “new” blocks, find a location on the disk to put it, then call the RAID commands to get it to store the data</li>

 <li>On writes for old blocks, figure out where you put it, then call the RAID commands to get it to overwrite the data</li>

 <li>On reads, return the previously stored data in those blocks</li>

 <li>Close the array when you are done</li>

</ol>

<h3>Honors Option</h3>

The tagline device driver should implement three different block allocation strategies. They should be defined by adding an enumerated type and associated variable that is used to tell the driver which allocation to use for which run. The allocation strategies include:

<ol>

 <li><code>TAGLINE_LINEAR</code> – this allocation strategy should allocate blocks on the first disk until full, then second, then third, and so on. The blocks on each disk should be allocated from lowest to highest, in order.</li>

 <li><code>TAGLINE_BALANCED</code> – this allocation strategy should allocate blocks evenly across the disks (round-robin). The blocks on each disk should be allocated from highest to lowest.</li>

 <li><code>TAGLINE_RANDOM</code> – this allocation strategy should allocate blocks randomly across the disks. The blocks on each disk should be allocated randmonly as well.</li>

</ol>

You need to modify the main program to accept run-time parameters “<code>--linear</code>“, “<code>--balanced</code>, and “<code>--random</code>“. The program should fail if more than one of these is provided, and the driver should be configured with the appropriate value if the parameter is provided. The driver should default to random.

<h2>Assignment Details</h2>

Below are the step by step details of how to implement, test, and submit your device drivers as part of the class assignment. As always, the instructor and TAs are available to answer questions and help you make progress. Note that this assignment is deceptively complicated and will likely take even the best students a substantial amount of time. Please plan to allocate your time accordingly.

<ol id="assignlist">

 <li>From your virtual machine, download the starter source code provided for this assignment. To do this, use the <code>wget</code> utility to download the file off the main course website:

  <center>

   <a href="http://www.cse.psu.edu/~mcdaniel/cmpsc311-f15/docs/assign2-starter.tgz">http://www.cse.psu.edu/~mcdaniel/cmpsc311-f15/docs/assign2-starter.tgz</a>

  </center></li>

 <li>Copy the downloaded file into your development directory and unpack it.<code>% cd cmpsc311</code><code>% tar xvfz assign1-starter.tgz</code>Once unpacked, you will have the following starter files in the <tt>assign1</tt> directory. All of your work should be done in this directory.</li>

 <li>You are to complete the <code>tagline_sim</code> program but implementing the four tagline abstraction functions described above. All of this code will be implemented in the source code file <tt>tagline_driver.c</tt>. You are free to create any additional functions that you see a need for, so long as they are all in the same code file.</li>

 <li>Add comments to all of your files stating what the code is doing. Fill out the comment function header for each function you are defining in the code. A sample header you can copy for this purpose is provided for the main function in the code.</li>

 <li>To test your program, you will run it with sample data I provide. To do this, run the program from the command line with sample input <code>sample-workload.dat</code> and <code>sample-workload2.dat</code>. E.g.,

  <center>

   <code>./tagline_sim -v sample-workload.dat</code><code>./tagline_sim -v sample-workload2.dat</code>

  </center>You must use the “-v” option before the workload filename when running the simulation to get meaningful output. Once you have your program running correctly, you see the following message at the end of the output:

  <center>

   <tt>[INFO] Tagline simulation completed successfully.</tt>

  </center></li>

</ol>


