# Overview

Often there is a need to temporarily save a file to 'disk' for the consumption of external tools. Or maybe you can pipe some input info to an external tool
but has no way of forcing such external tool to pipe its output straight to your software: **it wants to write a file to disk**.
Disk operations are slow and if repeated too often can shorten the lifespan of underlying media.

On Linux, most distributions offer a /tmp directory BUT it is on physical media. However, modern distributions often offer at east two
places where one can safely create temporary files in RAM: /dev/run/<uid>, /run/shm and /dev/shm

/dev/run/<uid> is ideal for your temporary files. It is writable and readable only by your user.
/dev/shm is usually world-readable and world-writable (just like /tmp), it is often used for IPC (inter process communication) but can also serve well as a temporary RAM-based tempdir

This module is very simple and tries not to reinvent the wheel. It will check /tmp to see if it in a ramdisk or not. And it will also check
if you have other options where to place your temporary files/dirs on a memory-based file system like tmpfs or ramfs.

With this info you are well served by pythons builtin modules and external packages like pathlib or pyfilesystem2 to move on to do your things.   

**To know more, I recommend the following links:**
https://unix.stackexchange.com/questions/162900/what-is-this-folder-run-user-1000
https://superuser.com/questions/45342/when-should-i-use-dev-shm-and-when-should-i-use-tmp


# API
This module searches for paths hosted on filesystems of type belonging to MEM_BASED_FS ('tmpfs' or 'ramfs').
Paths in SUITABLE_PATHS are searched and the first path found that exists and is stored on a filesystem whose type 
belongs to MEM_BASED_FS will be used as the tempdir.
If no suitable path is found, then if fallback = True, we will fallback to default tempdir (as determined by tempfile stdlib). If fallback is a path, then we will default to it.
If fallback is false, a RunTimeError exception is raised.

The MemoryTempfile constructor has arguments that let you change how the algorith works.
You can change the order of paths (with preferred_paths), add new paths to the seach (with preferred_paths and/or with additional_paths) 
and you can exclude certain paths (with removed_paths).
You can change the filesystem types you accept (with filesystem_types) and specify whether or not to fallback to a vanilla tempdir as a last resort.

Then, all methods available from tempfile stdlib are available through MemoryTempfile.

# Usage

## Example 1:

    from memory_tempfile import MemoryTempfile
    
    tempfile = MemoryTempfile()
    
    with tempfile.TemporaryFile() as 
        # as usual...
        
## Example 2:

    # We now do not want to use /dev/shm or /run/shm and no ramfs paths
    # If /run/user/{uid} is available, we prefer it to /tmp
    # And we want to try /var/run as a last resort
    # If all fails, fallback to platform's tmp dir
    
    from memory_tempfile import MemoryTempfile
    import memory_tempfile

    # By the way, all paths with string {uid} will have it replaced with the user id
    tempfile = MemoryTempfile(preferred_paths=['/run/user/{uid}'], remove_paths=['/dev/shm', '/run/shm'],
                              additional_paths=['/var/run'], filesystem_types=['tmpfs'], fallback=True)
    
    if tempfile.found_mem_tempdir():
        print('We could use any of the followig paths: {}'.format(tempfile.get_usable_mem_tempdir_paths()))
        print('And we are using now: {}'.format(tempfile.gettempdir()))
    
    with tempfile.NamedTemporaryFile() as ntf:
        # use it as usual...
        pass