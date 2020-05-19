# Files and Directories

CALLING LSEEK\(\) DOES NOT PERFORM A DISK SEEKby calling open\(\) and passing it the O CREAT flag, a program can create a new file.

**39.1 Files And Directories**

process: a virtualization of the CPU

address space: a virtualization of memory

Unlike memory, whose contents are lost when there is a power loss, a persistent-storage device keeps such data intact.

the low-level name of a file is often referred to as its inode number.

The directory hierarchy starts at a root directory \(in UNIX-based sys- tems, the root directory is simply referred to as /\)

![](.gitbook/assets/image%20%2811%29.png)

valid directories in the example are /, /foo, /bar, /bar/bar, /bar/foo and valid files are /foo/bar.txt and /bar/foo/bar.txt.

**39.2  The File System Interface**

by calling open\(\) and passing it the O CREAT flag, a program can create a new file.

```c
int fd = open("foo", O_CREAT|O_WRONLY|O_TRUNC, S_IRUSR|S_IWUSR);
```

the second parameter creates the file \(O CREAT\) if it does not exist, ensures that the file can only be written to \(OWRONLY\), and, if the file already exists, truncates it to a size of zero bytes thus removing any exist- ing content \(O TRUNC\). The third parameter specifies permissions, in this case making the file readable and writable by the owner.

```c
prompt> strace cat foo
...
open("foo", O_RDONLY|O_LARGEFILE) =3 
read(3, "hello\n", 4096) =6 
write(1, "hello\n", 6) =6 
hello
read(3, "", 4096) =0
close(3) =0
...
prompt>
```

![](.gitbook/assets/image%20%282%29.png)

![](.gitbook/assets/image%20%2815%29.png)

```text
off_t lseek(int fildes, off_t offset, int whence);
```

```bash
If whence is SEEK_SET, the offset is set to offset bytes.
If whence is SEEK_CUR, the offset is set to its current location 
plus offset bytes.
If whence is SEEK_END, the offset is set to the size of the 
file plus offset bytes
```

CALLING LSEEK\(\) DOES NOT PERFORM A DISK SEEK

The lseek\(\) call simply changes a variable in OS memory that tracks, for a particular process, at which offset its next read or write will start. A disk seek occurs when a read or write issued to the disk is not on the same track as the last read or write, and thus neces- sitates a head movement. Calling lseek\(\) can lead to a seek in an upcoming read or write, but absolutely does not cause any disk I/O to occur itself.

![](.gitbook/assets/image%20%281%29.png)

![](.gitbook/assets/image%20%286%29.png)

![](.gitbook/assets/image%20%283%29.png)

![](.gitbook/assets/image%20%2812%29.png)

**Shared File Table Entries: fork\(\) And dup\(\)**

```bash
int main(int argc, char *argv[]) {
    int fd = open("file.txt", O_RDONLY);
    assert(fd >= 0);
    int rc = fork();
    if (rc == 0) {
        rc = lseek(fd, 10, SEEK_SET);
        printf("child: offset %d\n", rc);
    } else if (rc > 0) {
        (void) wait(NULL);
        printf("parent: offset %d\n", (int) lseek(fd, 0, SEEK_CUR));
 }
return 0; }
```

```bash
prompt> ./fork-seek
child: offset 10
parent: offset 10
prompt>
```

```bash
int main(int argc, char *argv[]) {
    int fd = open("README", O_RDONLY);
    assert(fd >= 0);
    int fd2 = dup(fd);
    // now fd and fd2 can be used interchangeably
    return 0;
}
```

**39.7 Writing Immediately With fsync\(\)**

```text
int fd = open("foo", O_CREAT|O_WRONLY|O_TRUNC,
                S_IRUSR|S_IWUSR);
assert(fd > -1);
int rc = write(fd, buffer, size);
assert(rc == size);
rc = fsync(fd);
assert(rc == 0);
```

**39.8 Renaming Files**

```text
prompt> mv foo bar
```

Imagine that you are using a file ed- itor \(e.g., emacs\), and you insert a line into the middle of a file. The file’s name, for the example, is foo.txt. The way the editor might update the file to guarantee that the new file has the original contents plus the line inserted is as follows \(ignoring error-checking for simplicity\):

```text
int fd = open("foo.txt.tmp", O_WRONLY|O_CREAT|O_TRUNC,
                    S_IRUSR|S_IWUSR);
write(fd, buffer, size); // write out new version of file
fsync(fd);
close(fd);
rename("foo.txt.tmp", "foo.txt");
```

**39.9 Getting Information About Files**

stat\(\) or fstat\(\) get metadata

```text
struct stat {
dev_t     st_dev;         /* ID of device containing file */
ino_t     st_ino;         /* Inode number */
mode_t    st_mode;        /* File type and mode */
nlink_t   st_nlink;       /* Number of hard links */
uid_t     st_uid;         /* User ID of owner */
gid_t     st_gid;         /* Group ID of owner */
dev_t     st_rdev;        /* Device ID (if special file) */
off_t     st_size;        /* Total size, in bytes */
blksize_t st_blksize;     /* Block size for filesystem I/O */
blkcnt_t  st_blocks;      /* Number of 512B blocks allocated */
 struct time_t st_atim;  /* Time of last access */
 struct time_t st_mtim;  /* Time of last modification */
 struct time_t st_ctim;  /* Time of last status change */
 };
```

```text
prompt> echo hello > file
prompt> stat file
File: ‘file’
  Size: 6   Blocks: 8   IO Block: 4096   regular file
Device: 811h/2065d Inode: 67158084    Links: 1
Access: (0640/-rw-r-----) Uid: (30686/remzi)
  Gid: (30686/remzi)
Access: 2011-05-03 15:50:20.157594748 -0500
Modify: 2011-05-03 15:50:20.157594748 -0500
Change: 2011-05-03 15:50:20.157594748 -0500
```

```text
prompt> strace rm foo
...
unlink("foo")                           = 0
...
```

You can see these directories by passing a flag \(-a\) to the program ls:

```text
prompt> ls -a
./ ../
prompt> ls -al
total 8
drwxr-x--- 2 remzi remzi
drwxr-x--- 26 remzi remzi 4096 Apr 30 16:17 ../

```

**39.12 Reading Directories**

```text
int main(int argc, char *argv[]) {
    DIR *dp = opendir(".");
    assert(dp != NULL);
    struct dirent *d;
    while ((d = readdir(dp)) != NULL) {
        printf("%lu %s\n", (unsigned long) d->d_ino, d->d_name);
    }
    closedir(dp);
    return 0;
}
```

```text
struct dirent {
ino_t          d_ino;       /* Inode number */
off_t          d_off;       /* Not an offset; see below */
unsigned short d_reclen;    /* Length of this record */
unsigned char  d_type;      /* Type of file; not supported
               by all filesystem types */
char           d_name[256]; /* Null-terminated filename */
};
```

you can delete a directory with a call to rmdir\(\), rmdir\(\)has the requirement that the directory be empty \(i.e., only has “.” and “..” entries\) before it is deleted. If you try to delete a non-empty directory, the call to rmdir\(\) simply will fail.

**39.14  Hard Links**

```text
prompt> echo hello > file
prompt> cat file
hello
prompt> ln file file2
prompt> cat file2
hello
```

The file is not copied in any way; rather, you now just have two human-readable names \(file and file2\) that both refer to the same file.

```text
prompt> ls -i file file2
67158084 file
67158084 file2
prompt>
```

When unlink\(\) is called, it removes the “link” between the human-readable name \(the file that is being deleted\) to the given inode number, and decre- ments the **reference count**; only when the reference count reaches zero does the file system also free the inode and related data blocks, and thus truly “delete” the file.

**39.15 Symbolic Links \(soft link\)**

Hard links are somewhat limited: you can’t create one to a directory \(for fear that you will create a cycle in the directory tree\); you can’t hard link to files in other disk partitions \(because inode numbers are only unique within a particular file system, not across file systems\).

To create such a link, you can use the same program ln, but with the -s flag. Here is an example:

```text
prompt> echo hello > file
prompt> ln -s file file2
prompt> cat file2
hello
```

```text
prompt> stat file
 ... regular file ...
prompt> stat file2
 ... symbolic link ...
```

```text
prompt> ls -al
drwxr-x---  2 remzi remzi   29 May  3 19:10 ./
drwxr-x--- 27 remzi remzi 4096 May  3 15:14 ../
-rw-r-----  1 remzi remzi    6 May  3 19:10 file
lrwxrwxrwx  1 remzi remzi    4 May  3 19:10 file2 -> file
```

file2 是4 bytes因为file文件名短，文件名长则更大

```text
prompt> echo hello > alongerfilename
prompt> ln -s alongerfilename file3
prompt> ls -al alongerfilename file3
-rw-r----- 1 remzi remzi  6 May  3 19:17 alongerfilename
lrwxrwxrwx 1 remzi remzi 15 May  3 19:17 file3 ->
                                         alongerfilename
```

dangling reference：

```text
prompt> echo hello > file
prompt> ln -s file file2
prompt> cat file2
hello
prompt> rm file
prompt> cat file2
cat: file2: No such file or directory
```

**39.16 Permission Bits And Access Control Lists**

Imagine we have an unmounted ext3 file system, stored in device partition /dev/sda1, that has the fol- lowing contents: a root directory which contains two sub-directories, a and b, each of which in turn holds a single file named foo. Let’s say we wish to mount this file system at the mount point /home/users.

```text
prompt> mount -t ext3 /dev/sda1 /home/users
prompt> ls /home/users/ 
ab
```

instead of having a number of separate file systems, mount unifies all file systems into one tree, making naming uniform and convenient.



