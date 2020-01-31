# mercurial to git

Environment for use hg-fast-export.sh without brutal install hg on your device

## Preparation:
You can use this tool in two variants:

### use docker image bscheshir/codeception:hg-to-git:alpine3.11 from docker hub:

Create your composition file
```sh
mkdir docker-mercurial-to-git
cd docker-mercurial-to-git
echo "version: '3.7'
services:
  alpine:
    image: bscheshir/hg-to-git:alpine3.11
    volumes:
      - ./git-repo:/data/git-repo
      - ./hg-repo:/data/hg-repo
" >> docker-compose.yml
```

### build it manually:
```
git clone https://github.com/bscheshirwork/docker-mercurial-to-git.git
cd docker-mercurial-to-git
```
we can use your own changes for `Dockerfile` and `docker-compose.yml` as you wish. 

## Usage

For both case we can use it same:

1.Run container and go inside 

note: `git-repo` and `hg-repo` be create from `root` user on volume mapping phase
```sh
docker-compose run --rm alpine bash
cd hg
```
2.Download hg repo from container
```sh
hg clone <remote repo URL> /data/hg-repo
```
3.Work with authors (see instruction above) 
```sh
cd /data/hg-repo
hg log | grep user: | sort | uniq | sed 's/user: *//' > /data/authors
```
This will take a few seconds, depending on how long your project’s history is, and afterwards the /tmp/authors file will look something like this:
```sh
cat /data/authors
```
```sh
bob
bob@localhost
bob <bob@company.com>
bob jones <bob <AT> company <DOT> com>
Bob Jones <bob@company.com>
Joe Smith <joe@company.com>
```
In this example, the same person (Bob) has created changesets under four different names, one of which actually looks correct, 
and one of which would be completely invalid for a Git commit. Hg-fast-export lets us fix this by turning each line into a rule: 
"<input>"="<output>", mapping an <input> to an <output>. 
Inside the <input> and <output> strings, all escape sequences understood by the python string_escape encoding are supported. 
If the author mapping file does not contain a matching <input>, that author will be sent on to Git unmodified. 
If all the usernames look fine, we won’t need this file at all. In this example, we want our file to look like this:
```sh
mcedit /data/authors
```
```sh
"bob"="Bob Jones <bob@company.com>"
"bob@localhost"="Bob Jones <bob@company.com>"
"bob <bob@company.com>"="Bob Jones <bob@company.com>"
"bob jones <bob <AT> company <DOT> com>"="Bob Jones <bob@company.com>"
The same kind of mapping file can be used to rename branches and tags when the Mercurial name is not allowed by Git.
```

note: I can't see any reason to save this file outside the container. We can copy content of this file manually if needed.

3.Work with convert tools

The next step is to create our new Git repository, and run the export script:
```sh
git init /data/git-repo
cd /data/git-repo
/data/fast-export/hg-fast-export.sh -r /data/hg-repo -A /data/authors
```
The -r flag tells hg-fast-export where to find the Mercurial repository we want to convert, 
and the -A flag tells it where to find the author-mapping file (branch and tag mapping files are specified 
by the -B and -T flags respectively). 
The script parses Mercurial changesets and converts them into a script for Git’s "fast-import" feature 
(which we’ll discuss in detail a bit later on). This takes a bit (though it’s much faster than it would be over the network), 
and the output is fairly verbose:
```sh
bash-5.0# /data/fast-export/hg-fast-export.sh -r /data/hg-repo -A /data/authors
Loaded 4 authors
master: Exporting full revision 1/22208 with 13/0/0 added/changed/removed files
master: Exporting simple delta revision 2/22208 with 1/1/0 added/changed/removed files
master: Exporting simple delta revision 3/22208 with 0/1/0 added/changed/removed files
[…]
master: Exporting simple delta revision 22206/22208 with 0/4/0 added/changed/removed files
master: Exporting simple delta revision 22207/22208 with 0/2/0 added/changed/removed files
master: Exporting thorough delta revision 22208/22208 with 3/213/0 added/changed/removed files
Exporting tag [0.4c] at [hg r9] [git :10]
Exporting tag [0.4d] at [hg r16] [git :17]
[…]
Exporting tag [3.1-rc] at [hg r21926] [git :21927]
Exporting tag [3.1] at [hg r21973] [git :21974]
Issued 22315 commands
git-fast-import statistics:
---------------------------------------------------------------------
Alloc'd objects:     120000
Total objects:       115032 (    208171 duplicates                  )
      blobs  :        40504 (    205320 duplicates      26117 deltas of      39602 attempts)
      trees  :        52320 (      2851 duplicates      47467 deltas of      47599 attempts)
      commits:        22208 (         0 duplicates          0 deltas of          0 attempts)
      tags   :            0 (         0 duplicates          0 deltas of          0 attempts)
Total branches:         109 (         2 loads     )
      marks:        1048576 (     22208 unique    )
      atoms:           1952
Memory total:          7860 KiB
       pools:          2235 KiB
     objects:          5625 KiB
---------------------------------------------------------------------
pack_report: getpagesize()            =       4096
pack_report: core.packedGitWindowSize = 1073741824
pack_report: core.packedGitLimit      = 8589934592
pack_report: pack_used_ctr            =      90430
pack_report: pack_mmap_calls          =      46771
pack_report: pack_open_windows        =          1 /          1
pack_report: pack_mapped              =  340852700 /  340852700
---------------------------------------------------------------------

$ git shortlog -sn
   369  Bob Jones
   365  Joe Smith
```
That’s pretty much all there is to it. All of the Mercurial tags have been converted to Git tags, and Mercurial branches and bookmarks have been converted to Git branches. 


4.Can see result in `git-data` outside of container or in `/data/git-data` inside of container

Now you’re ready to push the repository up to its new server-side home:
```sh
$ git remote add origin git@my-git-server:myrepository.git
$ git push origin --all
```


(migration to git EN)[https://git-scm.com/book/en/v2/Git-and-Other-Systems-Migrating-to-Git]

(convert tool)[https://github.com/frej/fast-export]

(migration to git RU)[https://git-scm.com/book/ru/v2/Git-%D0%B8-%D0%B4%D1%80%D1%83%D0%B3%D0%B8%D0%B5-%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D1%8B-%D0%BA%D0%BE%D0%BD%D1%82%D1%80%D0%BE%D0%BB%D1%8F-%D0%B2%D0%B5%D1%80%D1%81%D0%B8%D0%B9-%D0%9C%D0%B8%D0%B3%D1%80%D0%B0%D1%86%D0%B8%D1%8F-%D0%BD%D0%B0-Git]
