# Intro

Rsync-K is a project conducted by USTCLUG in OSPP-2021. Its development begins with a wish to optimize I/O performance on HDDs during regular pulling from upstream with the existing attr-tree built by Rsync-huai.

The modification changes generator, a role and a process on the same host as the receiver. The generator generally has two things to do. First, it checks local basis files after the receiver receives file lists and forks itself to be a generator. Second, it cuts a file into chunks and computes checksums. To be more specific, the modification changes the generator's behavior during phase one.

# Implementation

One option is added: seperate-attrs, followed by which is path of attr-tree, assumed to be somewhere in fast storage like memory(tmpfs) or SSD.

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

# Acknowledgments

I want to thank OSPP for giving financial support to this project.

I want to give sincere appreciation to Jiahao Li, who chatted with me early about this project and helped me solve a string output issue. Also, Keyu Tao, the official mentor of this project and my friend in real life, plays a significant role in this project.

