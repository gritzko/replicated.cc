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
map     txt      of 12345+orig    to hello.txt
hop     Branch
diff    12345+orig
show    txt of `swarmdb new ct`
show    memo at RELEASE

map csv of 2345+orig to table.csv
map json of bigdoc
map 'bigdoc.json'
```

### Prepositions

```
as - name
at - version
since - version
of - object
on - branch/snapshot
off - branch
to - file
```

### Names

* BranchName
* VERSION
* SNAPSHOT
* object

### More

```
new ct as memo
map [txt of memo to] memo.txt
see map in memo.txt
see [HEAD on] BranchName
see [HEAD of] memo
see VERSION [on BranchName]
hop onto BranchName
name [objects|yarns|branches|stores|snapshots] on BranchName
new ct on BranchName as objectname
forget objectname ?! (the log stays)
drop BranchName
read [ct of] memo at VERSION
query '@123+orig:ct since 12345+orig'
assert '@123....'
read txt of memo
write txt of memo at VERSION << stdin
list [branches|objects|stores|names|tips] [on BranchName]
diff txt of memo at VERSION2 since VERSION1
diff memo.txt since VERSION
merge BranchName as NewBranch
fork [on BranchName] as Forked
show txt of memo
show @memo:txt
read log of memo since VERSION
read [yarn Branch] since VERSION
hash VERSION
head [of object] [on Branch]
map [dir of root$yarnid]
dedup Branch1 Branch2 Branch3 as BRANCH
write txt of memo at BASEVER << stdin
blend memo on OtherBrnch 
show txt of `blend memo and amend on OtherBrnch`
vv
as ~ name
at
on
off
```




