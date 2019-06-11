---
layout: post
title: "Appendix B InnoDB Source Code Distribution"
categories: computer
tags:  computer sql mysql official manual MySQLInternalsManual
author: "web"
source: "https://dev.mysql.com/doc/internals/en/files-in-innodb-sources.html"
---

* content
{:toc}


[MySQL Internals Manual](/doc/internals/en/)  /  InnoDB Source Code Distribution

The `InnoDB` source files are the best place to look for information about internals of the file structure that MySQLers can optionally use for transaction support. But when you first look at all the subdirectories and file names you'll wonder: Where Do I Start? It can be daunting.

Well, I've been through that phase, so I'll pass on what I had to learn on the first day that I looked at `InnoDB` source files. I am very sure that this will help you grasp, in overview, the organization of `InnoDB` modules. I'm also going to add comments about what is going on -- which you should mistrust! These comments are reasonable working hypotheses; nevertheless, they have not been subjected to expert peer review.

Here's how I'm going to organize the discussion. I'll take each of the 31 `InnoDB` subdirectories that come with the MySQL 5.0 source code in `\innobase` (on my Windows directory). The format of each section will be like this every time:

**\\subdirectory-name (LONGER EXPLANATORY NAME)**
-------------------------

  

**File Name**

**What Name Stands For**

**Size**

**Comment Inside File**

file-name

my-own-guess

in-bytes

from-the-file-itself

... My-Comments

For example:



 

`"\ha (HASHING)

|  File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---
|  ha0ha.c  |   Hashing/Hashing   |     8,145   Hash table with external chains

  Comments about hashing will be here.
"`

The "Comment Inside File" column is a direct copy from the first /\* comment \*/ line inside the file. All other comments are mine. After I've discussed each directory, I'll finish with some notes about naming conventions and a short list of URLs that you can use for further reference.

Now let's begin.

### **\\btr (B-TREE)**



 

|`File Name |  What Name Stands For    |     Size   |  Comment Inside File 
|---|---|---|---
| btr0btr.c  | B-tree / B-tree         |     82,400  | B-tree  
|  btr0cur.c  | B-tree / Cursor         |    103,233  | index tree cursor   
| btr0sea.c |  B-tree / Search      |        41,788  | index tree adaptive search   
| btr0pcur.c | B-tree / persistent cursor  | 16,720 |  index tree persistent cursor`   

If you total up the sizes of the C files, you'll see that \\btr is the second-largest file group in InnoDB. This is understandable because maintaining a B-tree is a relatively complex task. Luckily, there has been a lot of work done to describe efficient management of B-tree and B+-tree structures, much of it open-source or public-domain, since their original invention over thirty years ago.

`InnoDB` likes to put everything in B-trees. This is what I'd call a "distinguishing characteristic" because in all the major DBMSs (like IBM DB2, Microsoft SQL Server, and Oracle), the main or default or classic structure is the heap-and-index. In InnoDB the main structure is just the index. To put it another way: InnoDB keeps the rows in the leaf node of the index, rather than in a separate file. Compare Oracle's Index Organized Tables, and Microsoft SQL Server's Clustered Indexes.

This, by the way, has some consequences. For example, you may as well have a primary key since otherwise InnoDB will make one anyway. And that primary key should be the shortest of the candidate keys, since `InnoDB` will use it as a pointer if there are secondary indexes.

Most importantly, it means that rows have no fixed address. Therefore the routines for managing file pages should be good. We'll see about that when we look at the \\row (ROW) program group later.

### **\\buf (BUFFERING)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  buf0buf.c   |Buffering / Buffering |65,582|   The database buffer buf_pool
|  buf0flu.c   |Buffering / Flush     |29,583|   ... flush algorithm
|  buf0lru.c   |/ least-recently-used |27,515|   ... replacement algorithm
|  buf0rea.c   |Buffering / read      |21,504|   ... read`

There is a separate file group (\\mem MEMORY) which handles memory requests in general. A "buffer" usually has a more specific definition, as a memory area which contains copies of pages that ordinarily are in the main data file. The "buffer pool" is the set of all buffers (there are lots of them because InnoDB doesn't depend on the operating system's caching to make things faster).

The pool size is fixed (at the time of this writing) but the rest of the buffering architecture is sophisticated, involving a host of control structures. In general: when InnoDB needs to access a new page it looks first in the buffer pool; InnoDB reads from disk to a new buffer when the page isn't there; InnoDB chucks old buffers (basing its decision on a conventional Least-Recently-Used algorithm) when it has to make space for a new buffer.

There are routines for checking a page's validity, and for read-ahead. An example of "read-ahead" use: if a sequential scan is going on, then a DBMS can read more than one page at a time, which is efficient because reading 32,768 bytes (two pages) takes less than twice as long as reading 16,384 bytes (one page).

### **\\data (DATA)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  data0data.c |Data / Data           |15,344|   SQL data field and tuple
|  data0type.c |Data / Type            |7,417|   Data types`

This is a collection of minor utility routines affecting rows.

### **\\db (DATABASE)**

There are no .c files in \\db, just one .h file with some definitions for error codes.

### **\\dict (DICTIONARY)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  dict0dict.c |Dictionary / Dictionary |114,263| Data dictionary system
|  dict0boot.c |Dictionary / boot     |11,704|   ... booting
|  dict0crea.c |Dictionary / Create   |37,278|   ... creation
|  dict0load.c |Dictionary / load     |34,049|   ... load to memory cache
|  dict0mem.c  |Dictionary / memory    |7,470|   ... memory object creation`

The data dictionary (known in some circles as the catalog) has the metadata information about objects in the database --- column sizes, table names, and the like.

### **\\dyn (DYNAMICALLY ALLOCATED ARRAY)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  dyn0dyn.c   |Dynamic / Dynamic        |994|   dynamically allocated array`

There is a single function in the dyn0dyn.c program, for adding a block to the dynamically allocated array. InnoDB might use the array for managing concurrency between threads.

At the moment, the \\dyn program group is trivial.

### **\\eval (EVALUATING)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  eval0eval.c |Evaluating/Evaluating |17,061|  SQL evaluator
|  eval0proc.c |Evaluating/Procedures  |5,001|  Executes SQL procedures`

The evaluating step is a late part of the process of interpreting an SQL statement --- parsing has already occurred during \\pars (PARSING).

The ability to execute SQL stored procedures is an InnoDB feature, but MySQL handles stored procedures in its own way, so the eval0proc.c program is unimportant.

### **\\fil (FILE)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  fil0fil.c   |File / File          |118,312|   The low-level file system`

The reads and writes to the database files happen here, in coordination with the low-level file i/o routines (see os0file.c in the \\os program group).

Briefly: a table's contents are in pages, which are in files, which are in tablespaces. Files do not grow; instead one can add new files to the tablespace. As we saw earlier (discussing the \\btr program group) the pages are nodes of B-trees. Since that's the case, new additions can happen at various places in the logical file structure, not necessarily at the end. Reads and writes are asynchronous, and go into buffers, which are set up by routines in the \\buf program group.

### **\\fsp (FILE SPACE)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  fsp0fsp.c   |File Space Management |110,495|  File space management`

I would have thought that the \\fil (FILE) and \\fsp (FILE SPACE) MANAGEMENT programs would fit together in the same program group; however, I guess the InnoDB folk are splitters rather than lumpers.

It's in fsp0fsp.c that one finds some of the descriptions and comments of extents, segments, and headers. For example, the "descriptor bitmap of the pages in the extent" is in here, and you can find as well how the free-page list is maintained, what's in the bitmaps, and what various header fields' contents are.

### **\\fut (FILE UTILITY)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  fut0fut.c   |File Utility / Utility   |293|   File-based utilities
|  fut0lst.c   |File Utility / List   |14,176|   File-based list utilities`

Mainly these small programs affect only file-based lists, so maybe saying "File Utility" is too generic. The real work with data files goes on in the \\fsp program group.

### **\\ha (HASHING)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  ha0ha.c     |Hashing / Hashing      |8,145|   Hash table with external chains
|  hash0hash.c |Hashing / Hashing      |3,283|   Simple hash table utility`

The two C programs in the \\ha directory --- ha0ha.c and hash0hash.c --- both refer to a "hash table" but hash0hash.c is specialized, it is mostly about accessing points in the table under mutex control.

When a "database" is so small that InnoDB can load it all into memory at once, it's more efficient to access it via a hash table. After all, no disk i/o can be saved by using an index lookup, if there's no disk.

### **\\ibuf (INSERT BUFFER)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  ibuf0ibuf.c |Insert Buffer /       |91,397|   Insert buffer`

The words "Insert Buffer" mean not "buffer used for INSERT" but "insertion of a buffer into the buffer pool" (see the \\buf BUFFER program group description). The matter is complex due to possibilities for deadlocks, a problem to which the comments in the ibuf0ibuf.c program devote considerable attention.

### **\\include (INCLUDE)**

All .h and .ic files are in the \\include directory. It's habitual to put comments along with the descriptions, so if (for example) you want to see comments about operating system file activity, the place to look is \\include\\os0file.h.

### **\\lock (LOCKING)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  lock0lock.c |Lock / Lock           |139,207| The transaction lock system`

If you've used DB2 or SQL Server, you might think that locks have their own in-memory table, that row locks might need occasional escalation to table locks, and that there are three lock types: Shared, Update, Exclusive.

All those things are untrue with `InnoDB`! Locks are kept in the database pages. A bunch of row locks can't be rolled together into a single table lock. And most importantly there's only one lock type. I call this type "Update" because it has the characteristics of DB2 / SQL Server Update locks, that is, it blocks other updates but doesn't block reads. Unfortunately, `InnoDB` comments refer to them as "x-locks" etc.

To sum it up: if your background is Oracle you won't find too much surprising, but if your background is DB2 or SQL Server the locking concepts and terminology will probably confuse you at first.

You can find my online article about the differences between Oracle-style and DB2/SQL-Server-style locks at: [http://dbazine.com/gulutzan6.html](http://dbazine.com/gulutzan6.html)

Now here is a notice from Heikki Tuuri of InnoDB. It concerns lock categories rather than lock0lock.c, but I place it in this section because this is the place that people are most likely to look for it.

Errata notice about `InnoDB` row locks:



 
>
	 #define LOCK_S  4 /* shared */
	  #define LOCK_X  5 /* exclusive */
	...
	/* Waiting lock flag */
	  #define LOCK_WAIT 256
	/* this wait bit should be so high that it can be ORed to the lock
	mode and type; when this bit is set, it means that the lock has not
	yet been granted, it is just waiting for its turn in the wait queue */
	...
	/* Precise modes */
	  #define LOCK_ORDINARY 0
	/* this flag denotes an ordinary next-key lock in contrast to LOCK_GAP
	or LOCK_REC_NOT_GAP */
	  #define LOCK_GAP 512
	/* this gap bit should be so high that it can be ORed to the other
	flags; when this bit is set, it means that the lock holds only on the
	gap before the record; for instance, an x-lock on the gap does not
	give permission to modify the record on which the bit is set; locks of
	this type are created when records are removed from the index chain of
	records */
	  #define LOCK_REC_NOT_GAP 1024
	/* this bit means that the lock is only on the index record and does
	NOT block inserts to the gap before the index record; this is used in
	the case when we retrieve a record with a unique key, and is also used
	in locking plain SELECTs (not part of UPDATE or DELETE) when the user
	has set the READ COMMITTED isolation level */
	  #define LOCK_INSERT_INTENTION 2048
	/* this bit is set when we place a waiting gap type record lock
	request in order to let an insert of an index record to wait until
	there are no conflicting locks by other transactions on the gap; note
	that this flag remains set when the waiting lock is granted, or if the
	lock is inherited to a neighboring record */

Errata notice about `InnoDB` row locks ends.

### **\\log (LOGGING)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  log0log.c   |Logging / Logging     |86,043|   Database log
|  log0recv.c  |Logging / Recovery    |91,352|   Recovery`

I've already written about the \\log program group, so here's a link to my previous article: "How Logs work with MySQL and InnoDB": [http://www.devarticles.com/c/a/MySQL/How-Logs-Work-On-MySQL-With-InnoDB-Tables](http://www.devarticles.com/c/a/MySQL/How-Logs-Work-On-MySQL-With-InnoDB-Tables)

### **\\mach (MACHINE FORMAT)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  mach0data.c |Machine/Data           |2,335|  Utilities for converting`

The mach0data.c program has two small routines for reading compressed ulints (unsigned long integers).

### **\\mem (MEMORY)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  mem0mem.c   |Memory / Memory       |10,310|   The memory management
|  mem0dbg.c   |Memory / Debug        |22,054|   ... the debug code
|  mem0pool.c  |Memory / Pool         |16,511|   ... the lowest level`

There is a long comment at the start of the mem0pool.c program, which explains what the memory-consumers are, and how InnoDB tries to satisfy them. The main thing to know is that there are really three pools: the buffer pool (see the \\buf program group), the log pool (see the \\log program group), and the common pool, which is where everything that's not in the buffer or log pools goes (for example the parsed SQL statements and the data dictionary cache).

### **\\mtr (MINI-TRANSACTION)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  mtr0mtr.c   |Mini-transaction /    |12,620|   Mini-transaction buffer
|  mtr0log.c   |Mini-transaction / Log |8,090|   ... log routines`

The mini-transaction routines are called from most of the other program groups. I'd describe this as a low-level utility set.

### **\\os (OPERATING SYSTEM)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  os0file.c   |OS / File            |104,081|   To i/o primitives
|  os0thread.c |OS / Thread            |7,754|   To thread control primitives
|  os0proc.c   |OS / Process          |16,919|   To process control primitives
|  os0sync.c   |OS / Synchronization  |14,256|   To synchronization primitives`

This is a group of utilities that other modules may call whenever they want to use an operating-system resource. For example, in os0file.c there is a public InnoDB function named os\_file\_create\_simple(), which simply calls the Windows-API function CreateFile. Naturally the call is within an "#ifdef \_\_WIN\_\_ ... #endif" block; the effective routines are somewhat different for other operating systems.

### **\\page (PAGE)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  page0page.c |Page / Page           |51,731|  Index page routines
|  page0cur.c  |Page / Cursor         |38,127|  The page cursor`

It's in the `page0page.c` program that you'll learn as follows: index pages start with a header, entries in the page are in order, at the end of the page is a sparse "page directory" (what I would have called a slot table) which makes binary searches easier.

Incidentally, the program comments refer to "a page size of 8 kB" which seems obsolete. In `univ.i` (a file containing universal constants) the page size is now #defined as 16KB.

### **\\pars (PARSING)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  pars0pars.c |Parsing/Parsing      |45,376|  SQL parser
|  pars0grm.c  |Parsing/Grammar      |62,685|  A Bison parser
|  pars0opt.c  |Parsing/Optimizer    |31,268|  Simple SQL Optimizer
|  pars0sym.c  |Parsing/Symbol Table  |5,239|  SQL parser symbol table
|  lexyy.c     |Parsing/Lexer        |62,071|  Lexical scanner`

The job is to input a string containing an SQL statement and output an in-memory parse tree. The EVALUATING (subdirectory \\eval) programs will use the tree.

As is common practice, the Bison and Flex tools were used --- `pars0grm.c` is what the Bison parser produced from an original file named `pars0grm.y` (also supplied), and `lexyy.c` is what Flex produced.

Since `InnoDB` is a DBMS by itself, it's natural to find SQL parsing in it. But in the MySQL/InnoDB combination, MySQL handles most of the parsing. These files are unimportant.

### **\\que (QUERY GRAPH)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  que0que.c   |Query Graph / Query   |30,774|   Query graph`

The program que0que.c ostensibly is about the execution of stored procedures which contain commit/rollback statements. I took it that this has little importance for the average MySQL user.

### **\\read (READ)**



 


| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  read0read.c |Read / Read           |9,935|  Cursor read`

The `read0read.c` program opens a "read view" of a query result, using some functions in the \\trx program group.

### **\\rem (RECORD MANAGER)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  rem0rec.c   |Record Manager        |38,573|   Record Manager
|  rem0cmp.c   |Record Manager /      |26,617|   Comparison services for records
              Comparison`

There's an extensive comment near the start of rem0rec.c title "Physical Record" and it's recommended reading. At some point you'll ask what are all those bits that surround the data in the rows on a page, and this is where you'll find the answer.

### **\\row (ROW)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  row0row.c   |Row / Row             |18,375|   General row routines
|  row0uins.c  |Row / Undo Insert      |6,799|   Fresh insert undo
|  row0umod.c  |Row / Undo Modify     |19,712|   Undo modify of a row
|  row0undo.c  |Row / Undo            |10,512|   Row undo
|  row0vers.c  |Row / Version         |14,385|   Row versions
|  row0mysql.c |Row / MySQL          |112,462|   Interface [to MySQL]
|  row0ins.c   |Row / Insert          |42,829|   Insert into a table
|  row0sel.c   |Row / Select         |111,719|   Select
|  row0upd.c   |Row / Update          |51,824|   Update of a row
|  row0purge.c |Row / Purge           |15,609|   Purge obsolete records`

Rows can be selected, inserted, updated/deleted, or purged (a maintenance activity). These actions cause following actions, for example after insert there can be an index-update test, but it seems to me that sometimes the following action has no MySQL equivalent (yet) and so is inoperative.

Speaking of MySQL, notice that one of the larger programs in the \\row program group is the "interface between Innobase row operations and MySQL" (row0mysql.c) --- information interchange happens at this level because rows in InnoDB and in MySQL are analogous, something which can't be said for pages and other levels.

### **\\srv (Server)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  srv0srv.c   |Server / Server       |75,633|   Server main program
|  srv0que.c   |Server / Query         |2,463|   Server query execution
|  srv0start.c |Server / Start        |50,154|   Starts the server`

This is where the server reads the initial configuration files, splits up the threads, and gets going. There is a long comment deep in the program (you might miss it at first glance) titled "IMPLEMENTATION OF THE SERVER MAIN PROGRAM" in which you'll find explanations about thread priority, and about what the responsibilities are for various thread types.

`InnoDB` has many threads, for example "user threads" (which wait for client requests and reply to them), "parallel communication threads" (which take part of a user thread's job if a query process can be split), "utility threads" (background priority), and a "master thread" (high priority, usually asleep).

### **\\sync (SYNCHRONIZATION)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  sync0sync.c |Synchronization /    |37,940|  Mutex, the basic sync primitive
|  sync0arr.c  |... / array          |26,455|  Wait array used in primitives
|  sync0rw.c   |... / read-write     |22,846|  read-write lock for thread sync`

A mutex (Mutual Exclusion) is an object which only one thread/process can hold at a time. Any modern operating system API has some functions for mutexes; however, as the comments in the sync0sync.c code indicate, it can be faster to write one's own low-level mechanism. In fact the old assembly-language XCHG trick is in sync0sync.c's helper file, \\include\\sync0sync.ic. This is the only program that contains any assembly code.

The i/o and thread-control primitives are called extensively. The word "synchronization" in this context refers to the mutex-create and mutex-wait functionality.

### **\\thr (Thread Local Storage)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  thr0loc.c   |Thread / Local         |5,334|   The thread local storage`

`InnoDB` doesn't use the Windows-API thread-local-storage functions, perhaps because they're not portable enough.

### **\\trx (Transaction)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  trx0trx.c   |Transaction /         |50,480|   The transaction
|  trx0purge.c |Transaction / Purge   |29,133|   ... Purge old versions
|  trx0rec.c   |Transaction / Record  |37,346|   ... Undo log record
|  trx0roll.c  |/ Rollback            |31,448|   ... Rollback
|  trx0sys.c   |Transaction / System  |27,018|   ... System
|  trx0rseg.c  |/ Rollback segment     |6,445|   ... Rollback segment
|  trx0undo.c  |Transaction / Undo    |51,519|   ... Undo log`

`InnoDB`'s transaction management is supposedly "in the style of Oracle" and that's close to true but can mislead you.

*   First: `InnoDB` uses rollback segments like Oracle8i does — but Oracle9i uses a different name.
    
*   Second: `InnoDB` uses multi-versioning like Oracle does — but I see nothing that looks like an Oracle ITL being stored in the `InnoDB` data pages.
    
*   Third: `InnoDB` and Oracle both have short (back-to-statement-start) versioning for the `READ COMMITTED` isolation level and long (back-to-transaction-start) versioning for higher levels — but `InnoDB` and Oracle have different "default" isolation levels.
    
*   Finally: `InnoDB`'s documentation says it has to lock "the gaps before index keys" to prevent phantoms — but any Oracle user will tell you that phantoms are impossible anyway at the `SERIALIZABLE` isolation level, so key-locks are unnecessary.
    

The main idea, though, is that `InnoDB` has multi-versioning. So does Oracle. This is very different from the way that DB2 and SQL Server do things.

### **\\usr (USER)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  usr0sess.c  |User / Session         |1,740|   Sessions`

One user can have multiple sessions (the session being all the things that happen between a connect and disconnect). This is where `InnoDB` used to track session IDs, and server/client messaging. It's another of those items which is usually MySQL's job, though. So now usr0sess.c merely closes.

### **\\ut (UTILITIES)**



 

| `File Name |  What Name Stands For | Size   |  Comment Inside File
|---|---|---|---|
|  ut0ut.c     |Utilities / Utilities  |9,728|   Various utilities
|  ut0byte.c   |Utilities / Debug        |793|   Byte utilities
|  ut0rnd.c    |Utilities / Random     |1,474|   Random numbers and hashing
|  ut0mem.c    |Utilities / Memory    |10,358|   Memory primitives
|  ut0dbg.c    |Utilities / Debug      |2,579|   Debug utilities`

The two functions in ut0byte.c are just for lower/upper case conversion and comparison. The single function in ut0rnd.c is for finding a prime slightly greater than the given argument, which is useful for hash functions, but unrelated to randomness. The functions in ut0mem.c are wrappers for "malloc" and "free" calls — for the real "memory" module see section \\mem (MEMORY). Finally, the functions in ut0ut.c are a miscellany that didn't fit better elsewhere: get\_high\_bytes, clock, time, difftime, get\_year\_month\_day, and "sprintf" for various diagnostic purposes.

In short: the \\ut group is trivial.

This is the end of the section-by-section account of `InnoDB` subdirectories.

**Some Notes About Structures**
----------------

InnoDB's job, as a storage engine for MySQL, is to provide: commit-rollback, crash recovery, row-level locking, and consistent non-blocking reads. How? With locks, a paged-file structure with buffer pooling, and undo/redo logs,

The locks are kept in bit maps in main memory. Thus InnoDB differs from Oracle in one respect: instead of storing lock information on the page as Oracle does with Interested Transaction Lists, InnoDB keeps it in a separate and more volatile structure. But both Oracle and InnoDB try to achieve a similar goal: "writers don't block readers". So a typical InnoDB row-read involves: (a) if the reading is for writing, then check if the row is locked and if so wait; (b) if according to the information in the row header the row has been changed by some newer transaction, then get the older version from the log. We call the (b) part "versioning" because it means that a reader can get the older version of a row and thus will have a temporally consistent view of all rows.

The InnoDB workspace consists of: tablespace and log files. A tablespace consists of: segments, as many as necessary. A segment is usually a file, but might be a raw disk partition. A segment consists of: extents. An extent consists of: 64 pages. A page's length is always 16KB, for both data and index. A page consists of: a page header, and some rows. The page and row formats are the subjects of later chapters.

InnoDB keeps two logs, the redo log and the undo log.

The redo log is for re-doing data changes that had not been written to disk when a crash occurred. There is one redo log for the entire workspace, it contains multiple files (the number depends on innodb\_log\_files\_in\_group), it is circular (that is, after writing to the last file InnoDB starts again on the first file). The file header includes the last successful checkpoint. A redo log record's contents are: Page Number (4 bytes = page number within tablespace), Offset of change within page (2 bytes), Log Record Type (insert, update, delete, "fill space with blanks", etc.), and the changes on that page (only redo values, not old values).

The undo log is primarily for removing data changes that had been written to disk when a crash occurred, but should not have been written, because they were for uncommitted transactions. Sometimes InnoDB calls the undo log the "rollback segment". The undo log is inside the tablespace. The "insert" section of the undo log is needed only for transaction rollback and can be discarded at COMMIT time. The "update/delete" section of the undo log is also useful for consistent reads, and can be discarded when InnoDB has ended all transactions that might need the undo log records to reconstruct earlier versions of rows. An undo log record's contents are: Primary Key Value (not a page number or physical address), Old Transaction ID (of the transaction that updated the row), and the changes (only old values).

COMMIT will write the contents of the log buffer to disk, and put undo log records in a history list. ROLLBACK will delete undo log records that are no longer needed. PURGE (an internal operation that occurs outside user control) will no-longer-necessary undo log records and, for data records that have been marked for deletion and are no longer necessary for consistent read, will remove the records. CHECKPOINT causes -- well, see the article "How Logs Work On MySQL With InnoDB Tables".

**A Note About File Naming**
----------------

There appears to be a naming convention. The first letters of the file name are the same as the subdirectory name, then there is a '0' separator, then there is an individual name. For the main program in a subdirectory, the individual name may be a repeat of the subdirectory name. For example, there is a file named ha0ha.c (the first two letters ha mean "it's in subdirectory ..\\ha", the next letter 0 means "0 separator", the next two letters mean "this is the main ha program"). This naming convention is not strict, though: for example the file lexyy.c is in the \\pars subdirectory.

**A Note About Copyrights**
----------------

Most of the files begin with a copyright notice or a creation date, for example "Created 10/25/1995 Heikki Tuuri". I don't know a great deal about the history of `InnoDB`, but found it interesting that most creation dates were between 1994 and 1998.

**References**
----------------

*   Ryan Bannon, Alvin Chin, Faryaaz Kassam and Andrew Roszko. "InnoDB Concrete Architecture" [http://www.swen.uwaterloo.ca/~mrbannon/cs798/assignment\_02/innodb.pdf](http://www.swen.uwaterloo.ca/~mrbannon/cs798/assignment_02/innodb.pdf)
    
    A student paper. It's an interesting attempt to figure out `InnoDB`'s architecture using tools, but I didn't end up using it for the specific purposes of this article.
    
*   Peter Gulutzan. "How Logs Work With MySQL And InnoDB" [http://www.devarticles.com/c/a/MySQL/How-Logs-Work-On-MySQL-With-InnoDB-Tables](http://www.devarticles.com/c/a/MySQL/How-Logs-Work-On-MySQL-With-InnoDB-Tables)
    
*   Heikki Tuuri. "InnoDB Engine in MySQL-Max-3.23.54 / MySQL-4.0.9: The Up-to-Date Reference Manual of InnoDB" [http://www.innodb.com/ibman.html](http://www.innodb.com/ibman.html)
    
    This is the natural starting point for all InnoDB information. Mr Tuuri also appears frequently on MySQL forums.
    

  

[PREV](zlib-directory.html "Previous: The zlib Directory")   [HOME](index.html "Start")   [UP](index.html "Up")   [NEXT](ix01.html "Next: Index")




