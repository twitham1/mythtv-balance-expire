
==== NAME ====

    mythtv-balance-expire - move recording files to balance multiple disks


==== SYNOPSIS ====

     mythtv-balance-expire [--help | --man | options]
        --help      show full usage of options and exit
        --man       show the full manual page and exit
        --size      balance by file size rather than file count
        --delta=N   expiring count delta allowed between disks [5]
        --titles    append titles to all files, not just moving files
        --subtitles append subtitles to titles
        --verbose   show the commands that would be run, or are being run:
        --go        actually do the moves, else just show what would be done


==== DESCRIPTION ====

    mythtv-balance-expire trades recording files between multiple local
    disks in a mythtv server to better balance the files over recorded time.
    Why would you want to do this you ask?

    Say you have 6 months of recordings but your disk is full so you add
    another. The new disk now gets most new recordings since it has more
    free space. 6 months later both are full and recording new shows. But
    the original disk is now expiring files from 1 year ago while the new
    disk is expiring files only 6 months old. Mythtv's auto-expire list
    implies your are losing 1 year old recordings but this is misleading
    since the new disk must expire shows much younger.

    Or, say you have a small disk and a large one, both full and recording
    via auto-expire. And you disable auto-expire on shows you really want to
    see someday to make sure you don't lose them to auto-expire. If you save
    a significant amount of shows this way, you can eventually have the
    small drive nearly full of saved files. So the auto-expire to record a
    new show is from just a few days ago, even if the large disk is expiring
    shows over a year old!

    So, what you really need is for all disks to hold a similar amount of
    oldest expiring files. This does that by moving recordings among the
    disks.


==   Usage and output   ==

    Run with no arguments. This will take no actions and just show the
    expiring recordings and recommended moves.

    The output shows expiring recordings in time sorted order. Counts are
    indented per disk, so each count "column" is a different disk. The lines
    look like:

            SUM DELTA GB CHAN_YYYYMMDDHHMMSS.mpg /path/to/recording

:   SUM Number of expiring files or --size so far on this recording path,
        before placing the current file. Each path is indented to a unique
        column.

:   DELTA
        Current delta of the path with the most expiring content compared to
        the path with the least. When this value exceeds --delta, a move is
        triggered.

:   GB  Size in gigabytes of this expiring file.

:   CHAN_YYYYMMDDHHMMSS.mpg
        Recording file name: channel, time stamp and file extension. The
        lines are sorted by the YYYYMMDDHHMMSS time stamp of the recording.

:   /path/to/recording
        Current path to the recording file. The --title of the file is
        optionally shown.

    Suggested moves are shown when a file should move to produce a better
    balance. These lines look like this:

            < GB expirefile /too/many/path -> /too/few/path XXX free < title
            > GB keeperfile /too/few/path -> /too/many/path YYY free > title

    The expiring file is delimited by < and shown with its size in
    gigabytes. It should move from the left path with too many to the right
    path with too few as shown by the arrow. The resulting file count and
    free space of the destination location is shown. Finally the title (and
    optionally --subtitle) of the moving file is displayed.

    If the destination is now more full than the source, a trade is made. A
    file with auto-expire disabled (a permanent "keeper" file) is preferred.
    This keeper file is delimited by >. It is selected by size that would
    result in closest headroom on the disks. If no keepers are left, then a
    future expiring file is used for trade instead. When balancing to a disk
    with more free space, no trade is needed.

    A footer is provided which shows the end state of each disk. A notice is
    given if nothing has actually happened yet, see --go.

    If the suggested moves look correct, you can add --verbose to see the
    commands that will be run. For the successful copies, the original
    location will be removed. If this still looks good, simply add --go to
    actually do the given file moves. When using --go you must be root or
    the owner of the recording directories (mythtv) so that you have
    permission to move the files.


==== OPTIONS ====

:   --size  Balance by expiring file size rather than file count. This may
            become the default in the future if it proves to work better
            than file count.

:   --delta N
            Require each disk's expiring file count or size over time to be
            within N of all other disks. If a disk has > N more than
            another, this triggers a file move to maintain this delta <= N.
            Lower values produce a better balance while higher values need
            less moves. You can manipulate this value to see how many moves
            will be required and how they will affect disk headroom, before
            deciding to use --go. A high value like 9999 will simply show
            the current [im]balance with no moves. When used with --size,
            the units are gigabytes, otherwise the units are file counts.
            The default delta is 5.

:   --titles
            Show titles of all recording files. Without this option, titles
            are shown only for files that are moving.

:   --subtitles
            Append subtitles to all titles.

:   --verbose
            Show the commands that will be run, or are being run with --go.
            If STDOUT is a terminal, then rsync(1) will use its --progress
            option to show copy progress, even without this --verbose.

:   --go    Actually do the file moves with rsync(1), removing the original
            only if successful. This requires write permission to the local
            recording directories so you must be root or mythtv on the
            server. If this option is not given, then no actions are taken.


==== CAVEATS ====

    You must have ~/.mythtv/config.xml configured to read your database and
    must be root or mythtv on the server for write permission to move the
    files. As shown in --verbose, rsync(1) is required.

    This only works with local files and may be confused by multiple
    backends or remote paths. Patches welcome if you have such a setup.

    Of course moving dozens of files will take a long time, so be patient
    when using --go. The initial --go will need to move more files than
    later runs. Rsync(1) --progress is shown on the terminal.

    This does not update the database at all. Mythtv will automatically
    discover the new locations of the recordings. Allow time for this to
    happen between --go runs.

    If your disks are different sizes, it may not be possible to balance
    recent time. This is normal - you will balance that time later when you
    run this again in the future. In fact, it is a good idea to run this
    periodically; I do so once a month. If you really trust the actions, you
    might crontab(1) this with redirection to a log file.


==== AUTHOR ====

    Timothy D Witham <twitham@sbcglobal.net>


==== COPYRIGHT AND LICENSE ====

    Copyright 2017-2018 Timothy D Witham.

    This program is free software; you can redistribute it and/or modify it
    under the same terms as Perl itself.

