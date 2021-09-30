# Intro

Rsync-K is a project conducted during OSPP-2021 by USTCLUG. It is developed with wish to optimize I/O performance when regularly pulling from upstream with existed attr-tree built by Rsync-huai.



The modification is mainly done with generator. Generator, a role and a process on the same host as receiver, generally has two things to do. First, it checks local basis files after receiver receives file list and fork it self to be a generator. Second, it cuts a file to chunks and computes checksums. To be more specific, the modification is done by changing generator's behavior during phase one.

# Implementation

One option is added: seperate-attrs, followed by which is path of attr-tree, assumed to be somewhere in memory (tmpfs).

# Test

Take CTAN repository to do tests.

### Origin

```bash
$ sudo bash -c "echo 3 > /proc/sys/vm/drop_caches"
$ ./rsync -avP --only-send-attrs /srv/repo/CTAN/ /srv/repo/CTAN-attr
```

Result (read from /proc/${generator_pid}/io):

```bash
read_bytes: 164126720
write_bytes: 0
```

### Rsync-K

First, generate attr-tree. The attr-tree is stored in tmpfs.

```bash
$ ./rsync -avP --only-send-attrs /srv/repo/CTAN/ /dev/shm/CTAN
```

Then, test.

```bash
$ sudo bash -c "echo 3 > /proc/sys/vm/drop_caches"
$ ./rsync -avP --seperate-attrs /dev/shm/CTAN /srv/repo/CTAN/ /srv/repo/CTAN-cp
```

Result (read from /proc/${generator_pid}/io):

```
read_bytes: 286720
write_bytes: 0
```

And `/srv/repo/CTAN-cp` should be an empty directory because all attributes in attr-tree are consistent with original repository.

As assumed, the bytes read has been enormously reduced, by 99.83%.

# Acknowledgements

I would like to thank OSPP for giving financial support to this project. 

I would like to give sincere appreciation to Jiahao Li, who chatted with me at an early time about this project and helped me to solve a string output issue, and Keyu Tao, who is officially mentor of this project and also my friend in real life.

