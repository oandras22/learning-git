# Learning Git 

My notes on book _Professional Git_

Created: 2020-01-25  |  Updated: 2020-02-15

## Chapter 2 - Key Concepts

### Design Concepts: User-facing

#### Centralized Model (CVCS)
Local machine - Central server
Operations: check-out, check-in


#### Distributed Model (DVCS)
Local Environment - Remote Repository
--> Local Repository 

Operations:
clone, src mgmt operations locally, push

Users do not have to be connected to the remote all the time


### Design Concepts: Internal

#### Delta Storage

- Used in a traditional source management systems
- Content is managed on a file-by-file basis
- a patch set (delta) created at each check-in for every file
- after 1 check-in: current version =  previous version + delta
- after many check-ins: current version = original vers + all the deltas
- perf. degaradation when change history is long (many deltas)

#### Snapshot Storage

- Used by Git
- Tracks revisions at directory level. revision: a slice of a directory tree structure at a point in time -- a snapshot
- When a commit is made into a Git repository a snapshot is taken of the workspacea at that point in time. Full content is recorded (not coputing deltas). Unchanged files are not copied but linked.
- How Git try to reduce storage space?
  - Links are used when file doesn't change
  - Content is compressed using zlib
  - packing: during garbage collection Git packs similar revisions in a compressed pack file.

#### Repository design considerations
##### Repository Scope
- Git works well with smaller directories --> preferably code should be structured to align this (set of many, smaller repositories)

##### File Scope

Phenomena: Git (and SMSs in general) doesn't create deltas between verrsions of binary files --> stores full versions for each change to a binary file --> can quickly consume significant disk space if the files are large

For Large files (>100MB text file; >10-20MB? binary) 
 - use a separarete repository (more convenient solution) OR
 - artirfact repository (e.g.: Nexus) OR
 - extensions managing large files: Git-LFS, Git-annex

##### Generated Content

 - In a model where generated files are stored in the source repository, if the source changes often, then the generated content must also be updated frequently in the repo. ==>
 - Ideally generated content should be placed into an artifact repository

##### Shared Code

Sometimes it is needed to share code from one repository to another. Possible solutions:
 - Using submodules. A submodule is a static reference to another repository that resides in the local environment. Hard to keep in sync --> not recommended for beginers
 - Using subtrees. 
 - build needed artifacts ifrom other repositories separately and specifí them as compoile-time or run-time dependencies to pull them in
___

## Chapter 3 - The Git Promotion Model

### The Levels of Git (Git Promotion Model)

The  different levels that users encounter when working with Git represents the stges that the contentmoves through from local development directory to server-side (remote) repository.
One way to think about these levels is to compare to the dev-test-prod modell:

| Dev | ==> | Test | ==> | Prod | ==> | Public |

| Working Directory | ==> | Staging Area | ==> | Local Repository | ==> | Remote Repository |

#### Working Directory

 DEF.: Content created, edited, deleted. Any new content must exist here before it can be put into Git (can be tracked).

- any directory or directory tree on the local system can be a working directory for a Git repository
- a working directory can ahave any number of subdirectories that form an overall _workspace_ (also referred as _working tree_ or _worktree_)

- When one connects Git to a local directory tree, by default Git creates a repository skeleton in special subdirectory at the top level of the tree (named .git). That repository skeleton is the local repository is the _local repository_.

 - .gitignore file: tells Git what should be not tracked (definied by list of files, directories and regexps)

#### Staging Area

DEF.: Holding area to accumulate and stage changes from working direcctory before they go further to local repository. It is a place to build up a set of content 

- it allows finer-grained control over the set of things that make up a change
- Two ways of viewing the utility of the staging area: user-initiated and Git-initiated

##### Utility of staging area - cases of user-initiated operations

The Prepare Scenario

As a user completes changes in his workspace he moves files that are ready into the staging area. Example: large checklist of files to  modify ... one do it in batches ... files in ready batches are moved to stagig area.


The Repair Scenario

Previous commit is _amended_ with any content that is in the staging area:
 - make any updates to the working directory
 - Put the updates into the staging area
 - Run the commit with the _amend_ option. This will cause the previous commit updated the with whatever is in the staging area and then overwrite the previous version of the commit in the local repository


##### Utility of staging area - when Git uses it

In case where there are _merge conflicts_, Git puts those files in the working directory for the user to fix, and stages any files that merged cleanly.

Other names of the staging area: index, chache.

#### Local Repository

DEF.: the actual source repository where content that Git manages is stored

- it is a source repository exclusively for the current user

#### Remote Repository

DEF.: a separate Git repository intended to collect and host content pushed to it from one or more local repositories. A place to share and access contenct from multiple users.

 - it is unique: Git does not make or use multiple copies of remote repository in th e server.
 - it can be cloned as many times as needed to separte local repositories. When users push changes from their local repositories, they are pushing them into the single corresponding remote repository that the local repositories were copied from.
- it does not make user-facing modifications to content, such as resolving conflicts for merging.

Local environment = working dir. + staging area + local repo

![Git in one picture](https://github.com/oandras22/learning-git/blob/master/Git_in_one_picture.jpg "Git in one pic")

![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")



___
## Chapter 4 - Configuration and Setup

### Git command syntax and format

    git <git-options> <command> <command-options> <operands>
    e.g.:
    git --working-tree=some_path
    git push
    git commit -m "my comment"
    git add *.py

Most common operand types: 
 - a) SHA1 commit ID | branch | tag
 - b) filenames/file-paths

Use '--'  if separation of two types of operand can be ambigous, e.g.:

    git <command> my-tag-name -- file_name

Autocomplete works for Git commands and their arguments.

### Porcelain and Plumbing Commands

#### Porcelain Commands 
 - high level commands 
 - based on pumbling commands
 - aggregate the functionality of plumbing commands and certain options and sequences
 - user-facing, more commonly used, more convinient
 - examples: add, push, pull, diff, status


#### Plumbing Commands 

 - lower level commands
 - not expected to be used by the average user
 - their purpose is to extract or modify content and information _directly_ from the repository
- examples: cat-files,ls-files,merge-base,show-ref


### Configuring Git

Syntax of git config :

    git config --<local|global|system> <parameter_group>.<parameter> <value>
    e.g.:
    git config --global user.name "Joe Geek"

#### Settings

Configure at least these global settings: user.name, user.email

Levels of configuration:

System
 - applies to all repositories all users
 - usually stored in a gitconfig file under `/user/etc` or `/usr/local/etc`
 
 Global
 - applies to all repositories for a particular user
 - stored in a file named `.gitconfig` in each user's home directory

Local
 - applies in the context of one repository
 - settings reside in `.git/config` in the local repository 
 - if no scope is specified in the `git config` command that means local is supposed by Git


##### Settings Hierarchy

Git uses a search order to find a setting: `local --> global --> system`.  If it finds a specific value in this search order that value is used. Beyond that, the union of all levels forms a superset of configuration values (unique local + unique global + unique system)


##### Configuration related commands

Note, if You omit the scope it means 'local'.

List settings:

    git config --list --[local|global|system]
    #show the origin of setting (=scope)
    git config --show-origin --list


See a value of a setting:

    git config --[local|global|system] <setting>

Set the value of a setting:

    git config --[local|global|system] <setting> <value>

Unset a setting (remove):

    git config --unset --[local|global|system] <setting>
    e.g.:
    git config --unset --global user.name

    
One-shot configuration (for the current operation):

    git -c <configuration setting>=<value> <rest of command line>


##### End of line settings

Two options controlled by the EOL setting:
 - How line endings stored in the content when it is committed onto the repository
 - How line endings are updated (or not) when content is checked out of the repository

1. **core.autocrlf=true** normalize line endings when storing files and insert CRLFs when files are checked out. Recommended setting on Windows

2. **core.autocrlf=input** normalize line endings when storing files but not to change anything when files are checked out. Recommended setting on Linux

3. **core.autocrlf=false** tells Git not to change anything when storing files or files are checked out. 

Best practice to set `core.autocrlf` to 'true' or 'input' depending on the environment.

Another good solution to set `core.autocrlf` setting in .gitconfig file.


##### Aliases

Aliases can be created this way:
    
    git config <scope> alias.<alias_name> <command string>
    e.g.:
    git config --global alias.prettylog log --pretty=format:"%h %ad | %s%d [%ad]" --graph --date=short

usage:
    
    git <alias_name>


#### What's in the Repository

`A Git repository is a content-addressable data store.` User can get out any content of it by using the hash value what was calculated when content was put into it.

HEAD file: keeps track of current branch the user currently works with

config file: configuration file of the local repository

object and pack directories: areas where content is actually stored

### Advanced Topics

Fig.4.3

Files and directories first exist as normal OS items on disk in the working directory.


































































