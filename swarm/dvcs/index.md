
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

see [CLI sketches](cli)
