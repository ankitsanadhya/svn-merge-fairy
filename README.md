# Introduction #

The Merge Fairy is a Python script that automates the process of
merging changes from one Subversion branch to another, based on an XML
configuration file that describes branches and their dependencies. For
example, you might want bug fixes from the release branch to be
automatically merged to the trunk branch.

In the event of a merge conflict or a build failure after merging, the
Merge Fairy sends email requesting help from a human to make a manual
merge, resuming automated merging once this done.

I originally wrote the Merge Fairy at Jobster, where it has been used
for the past few years. We also use it at Mergelab.

Follow the steps below to configure the Merge Fairy.

# Details #

## Configuring the Merge Fairy ##

### Enlist branches ###
Enlist each of the branches you want to merge between in the depot
subdirectory.

Each branch should have a script in the root directory called build
that should output "BUILD SUCCEEDED" if the build should be considered
good and "BUILD FAILED" if the build failed.

### Configure fairy.xml ###
Copy fairy.xml.example to fairy.xml and edit it to match your
configuration.

The branches subsection lists each of the branches that the Merge fairy
should know about.  The initial values of lastBuildVersion and
lastSyncVersion should be the most recent revisions in the
branch. For example:
```
        <branch url="server/branches/2008_03_04_attribution" path="server/branches/200\
8_03_04_attribution" buildSucceeded="true" lastBuildVersion="1328" lastSyncVersion="13\
28" />
```
The dependencies subsection describes relationships between branches.
For example, the trunk branch below should have changes automatically
merged from the attribution branch.  Set lastGoodMerge initially to
the most revision in the baseBranch.
```
        <dependency
            baseBranch="server/branches/attribution"
            dependentBranch="server/trunk"
            lastGoodMerge="1328" />
```
You will need to edit fairy.xml and restart the daemon whenever you add new branches that should be managed by the fairy.

### Configure fairyconfig.py ###
Copy fairyconfig.py.example to fairyconfig.py and edit it to match
> your configuration.  This file contains the email to notify when a
> merge fails, for example.

## Running the Merge Fairy ##

Run the start\_fairy script to start the Merge Fairy as a daemon.  Run
stop\_fairy to stop it.  On starting up, you should not see errors on
the console or in the /var/log/merge-fairy.err log file.

You can see the current state of the fairy by hitting
http://localhost:8081.

Other log files that help you track the fairies progress are
/var/log/merge-fairy.log and /var/log/merge-fairy.build.

You can verify correct operation of the fairy by checking a file into
the base branch. Assuming it merges cleanly, the Merge Fairy will
automatically check it the derived branch (e.g. trunk) and send an
email. If it fails to merge cleanly or build, the fairy will send an
email saying that a manual merge is required and block further
automated merging between those branches until this is done.

There are two directives that can be embedded in checkin mail to
communicate with the fairy.

**MERGES 1001** says that this checkin manually merges [revision 1001](https://code.google.com/p/svn-merge-fairy/source/detail?r=1001); you
should use this directive as instructed when the automatic merge
fails.

**NOFAIRY** says that the Merge Fairy should not attempt to
merge this checkin to any derived branches.  Use this (for example)
when backporting a change from a derived branch to the base branch.