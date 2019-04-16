
# RON-based revision control system

SwarmDB is a proof-of-concept RON-based CRDT-native database.
As such, it has well-defined versioning and synchronization
mechanisms. It was thus tempting to try using it as a
revision-control system. The key question here is whether
op-based CRDTs and RON could in principle address the pains
every *git* user is so familiar with.

Another reason to focus on such a demo was *dogfooding*.
Virtually every developer has strong opinions on revision
control systems. Hence, all the strong and weak sides of the
technology will be clear. Ultimately, the project's success or
failure will be obvious to everyone.

Again, as a PoC on top of a PoC, *swarmdb* is certainly not a
"git killer", so we do not focus on feature-by-feature
comparison. Instead, we focus on the new abilities and the use
cases swarmdb enables.

This text mostly skips the discussion of RON/CRDT inner
machinery. We focus on "Why" not "How". Please check the
references for a technical deep-dive.

For now, let's think how CRDTs help. Then, let's revisit this
text in half a year (1 Oct 2019) to see how this experiment
worked out.


## Simplified model

The bulk of git's complexity comes from the multitude of
"buckets" it has for the content: branches, commits, staging,
stash, working copy.  Moving your things between those buckets
is both tedious and error-prone. Especially, if you actually use
many buckets at once (e.g. stash, staging and several branches).
That is where most of the SNAFUs happen.  Well, historically git
made branching/merging a *relatively* frictionless process.
Still, CRDT may improve on that greatly.

Those buckets are necessitated by the underlying model: it is
binary blobs all the way down.  Developers think in patches. The
actual on-disk storage deals with deltas; it can't be efficient
otherwise. Still, the git's model is an immutable tree of binary
blobs which get created by commits and merges. That causes a
certain impedance mismatch.  Also, the fact a merge *creates*
new content certainly adds some friction. Git merges are not
algebraic!

SwarmDB employs an op-based model where everything is made of
small immutable ops. Most of the time, 1 op == 1 letter.
Formally, all your work is a linear op log (a *yarn*).  Your
collaborators create their yarns too. Ops reference each other,
in a causally consistent way (new to old).  A causally
consistent transitive closure of a yarn (a yarn and everything
it references) forms a *branch*. That is the only kind of a
"bucket" you have to deal with. In SwarmDB, everything is a
*branch*.  

As immutable CRDT ops merge deterministically, the data moves
between buckets *frictionlessly*.  Also, there is no impedance
mismatch between patches/deltas and blobs/snapshots. These are
simply groupings of ops; everything is made of ops here.

This simplification may greatly improve UX and maybe even enable
significant non-developer use of the technology.


## Shorter-timescale collaboration

A classic revision control system enables collaboration on the
"email timescale".  You make changes, you create a commit, your
push it.  A commit is a "big thing".  SwarmDB is a letter-precise
revision control technology; it may enable collaboration on the
"chat timescale".  There are no separate "commits" in this model.
In SwarmDB, *a version is just a number* referencing an
arbitrary point in the project's history.  

Hence, sending your changes over is not a special transaction
you have to prepare for. You may do it in real time, if you
wish. Or you may take the current version UUID and post it in
the chat.  (It has to be noted that CRDT-based real-time
collaboration was already implemented in various projects,
including Atom Teletype and one project by @gritzko.)

Fundamentally, the causal graph of RON operations is the same
class of DAG structure git is using, except RON ops are much
much finer grained and lack unnecessary friction; you don't have
to re-pack your edits several times over just to show them to
your colleague.


## IDE/editor/app integration

Finally, the third goal is to experiment with deep IDE/editor
integration. This assumes an asynchronous architecture for the
IDE. In this domain, Xi and Atom X-Ray projects must be
mentioned (remarkably, both projects use CRDTs).  Also, the
Language Server Protocol is highly relevant.  The underlying
vision is that a modern IDE has to run so many things at once,
it inevitably becomes a distributed system.

Ideally, the only synchronous part of it must be the renderer;
the rest may work in separate threads, processes, or remotely on
a server/cluster/cloud.  That might be grossly beneficial.
First, it may tap into bigger computational resource pool than a
typical laptop has.  Second, it may avoid unnecessary
duplication of work when your colleague's laptops are doing the
same work as yours.

This general idea might be seen as an inevitable evolutionary
step, although its implementation proves to be rather difficult.
Also, it broadens the scope of the project as it needs more
than source code revision control.  An IDE has to work with
numerous annotation overlays: syntax highlighting, linting,
compiler's hints, comments and issues.  Both LSP and xi
protocols are JSON RPC.

On the good side, SwarmDB synchronizes data structures, not
binary blobs.  That enables more meaningful behavior; it may
merge tables as tabls, lists as lists, text as text.
Potentially, that also allows to synchronize those overlays.
It is equally important that SwarmDB does not rely on the file
system as its interface; it may speak RON or JSON or plain text.

Ultimately, we speak of a vision of a real-time data/event bus
keeping that distributed IDE in sync.

This part of the project is purely exploratory.


----------------------------------------------------------------

## CLI interface sketches

### Sketch #1

```bash
# Create a replica and a yarn
./swarmdb create
1jQ7EX+l2KS73sKLs
# it prints the UUID for the initial (empty) version;
# yarn id is thus l2KS73sKLs


# Add a peer yarn, nickname it "joe"
./swarmdb peer ws://10.20.30.40/9Kk4Hekcdw as joe
Connected: project 'swarmdb CLI' branch 9Kk4Hekcdw by joe@domain.com
# Include its contents
./swarmdb see joe

# Get some files, use a nickname set by joe
./swarmdb mount sketches$joe map txt as sketches.txt
vi sketches.txt
# commmit
./swarmdb see sketches.txt

# What Has Been Seen Cannot Be Unseen
# ...but we want to reconsider
./swarmdb fork joe as redo
1jZu7a+8G2KeSk6f2
./swarmdb active redo
# well, try again
vi sketches.txt
# seemingly, I like it.
./swarmdb see sketches.txt
# copies the "redo" branch to the joe's repository
# this does not merge it anywhere yet, it is up to joe
./swarmdb push ws://10.20.30.40/
```

### Sketch 2

```bash
$ swarmdb create as One
You started a yarn
1kE7sJ0w28+gYpLcnUnF6

$ swarmdb new ct as hello
Created a new Causal Tree object tagged hello
1kE82u+gYpLcnUnF6

$ swarmdb map hello.txt
*ct #1kE82u+gYpLcnUnF6 is mapped to hello.txt (by the txt mapper)

$ echo Hello > hello.txt

$ swarmdb see hello.txt
seen changes to hello.txt
1kE85B0005+gYpLcnUnF6

$ swarmdb see hello.txt
see no changes
1kE85B0005+gYpLcnUnF6

$ swarmdb fork as Another
forked a yarn tagged Another off 1kE85B0005+gYpLcnUnF6
1kE8Im1892+i0GQOo9VsK

$ cat hello.txt
Hello

$ echo "Hello world" > hello.txt

$ swarmdb see
seen changes to hello.txt
1kE8Pp0005+i0GQOo9VsK

$ swarmdb hop One
hopped to the branch One
1kE85B0005+gYpLcnUnF6

$ cat hello.txt
Hello

$ echo "Hello beautiful" > hello.txt

$ swarmdb see
seen changes to hello.txt
1kE8Ze0009+gYpLcnUnF6

$ swarmdb merge Another as Both
merged 1kE85B0005+gYpLcnUnF6 and 1kE8Ze0009+i0GQOo9VsK
1kE8hv4QqQ+JnlcB2j5IT

$ cat hello.txt
Hello beautiful world

$ swarmdb list branches
1kE8Pp0005+i0GQOo9VsK    Another
1kE8Ze0009+gYpLcnUnF6    One
1kE8hv4QqQ+JnlcB2j5IT    Both

$ swarmdb query txt of 1kE8Ze0009+gYpLcnUnF6
Hello beautiful

$ swarmdb recover 1kE8Pp0005+i0GQOo9VsK as ANOTHER0LD
Recovered version 1kE8Pp0005 of Another
1kE8Pp0005+i0GQOo9VsK

$ swarmdb query txt of hello
Hello world

$ swarmdb list snapshots
1kE8Pp0005+i0GQOo9VsK   ANOTHER0LD

$ swarmdb list objects
1kE82u+gYpLcnUnF6       hello

```

### Sketch 3

full command grammar

```
create  Branch
fork    Branch   off Branch
see     'file.txt'
see     Branch
see     12345+yarn
map     txt      of 12345+orig    as hello.txt
hop     Branch
diff    12345+orig
show    txt of `swarmdb new ct`

map csv of 2345+orig to table.csv
map json of bigdoc
map 'bigdoc.json'
```
