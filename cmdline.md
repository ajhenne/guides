
# Table of Contents

1.  [Programs](#org73d929c)
    1.  [tar](#orgccd3603)
    2.  [tmux](#org77eb4eb)
2.  [Useful things to do](#org9b661bf)
    1.  [ls all files in directory except](#org876aedb)
    2.  [List all python files available as commands in path](#org832cf62)
    3.  [Loop look for a file every 10 seconds until found, then tail -f file](#orge5c7a03)

Basic instructions for cmdline utilities that are hopefully useful.


<a id="org73d929c"></a>

# Programs


<a id="orgccd3603"></a>

## tar

`tar -czvf name-of-archive.tar.gz /path/to/directory-or-file`

Here&rsquo;s what those switches actually mean:

-c: Create an archive.
-z: Compress the archive with gzip.
-v: Display progress in the terminal while creating the archive, also known as &ldquo;verbose&rdquo; mode. The v is always optional in these commands, but it&rsquo;s helpful.
-f: Allows you to specify the filename of the archive.

Once you have an archive, you can extract it with the tar command. The following command will extract the contents of archive.tar.gz to the current directory.

`tar -xzvf archive.tar.gz`


<a id="org77eb4eb"></a>

## tmux

Create new session.
`tmux new -s <session_name>`

Detatch from session.
`ctrl+b d`

List sessions.
`tmux ls`

Attatch to last session, or specific session.
`tmux a`
`tmux a -t <session_name>`

Kill a session.
`tmux kill-session -t <session_name>`

<https://tmuxcheatsheet.com>


<a id="org9b661bf"></a>

# Useful things to do


<a id="org876aedb"></a>

## ls all files in directory except

Ignore just a single file, or ignore all files matching a wildcard.
`ls -I file.txt`
`ls -I "*.txt"`


<a id="org832cf62"></a>

## List all python files available as commands in path

find $(echo $PATH | tr &rsquo;:&rsquo; &rsquo; &rsquo;) -name &ldquo;\*.py&rdquo; -type f -executable 2>/dev/null


<a id="orge5c7a03"></a>

## Loop look for a file every 10 seconds until found, then tail -f file

    #!/bin/bash
    
    while true; do
        FILE=$(ls slurm*.out 2>/dev/null | head -n 1)
    
        if [ -n "$FILE" ]; then
            echo "FILE FOUND"
            tail -f "$FILE"
            break
        else
            sleep 10
        fi
    done

