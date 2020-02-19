# sync_ssh_keys
Set of scripts to help sync SSH public keys from a central server

After reading various sources on best practice for ssh keys, one of the take aways was that each machine should have it's own key.  A single key should not be used across multiple machines.  The idea here is to minimize impact should a key be compromised.  Now my purpose here is not to debate the finer points of ssh key management nor explore various key exploitation scenarios.

I have ~70 VM's that I maintain on a personal level.  Many for my ham radio hobby, others for home infrastructure, etc.  And out of these ~70 machine I'd say ~60 are linux or BSD based.  So all of a sudden I have a significant amount of SSH keys to think about managing.  Not to mention I may bounce from machine to machine and not always start from a central point, where a single key would simplify things.

Add to that scenario, that I have from time to time fellow hams who come and go from various admin tasks.  I thought it would be nice to be able to sync not only my keys but their keys as well.  So that's what lead to this project.

Right now the code is pretty rough, but functional.  I do plan to clean things up, but at this point do not plan to change much in the theory of operation.

Here's the low down:

LICENSE - Your standard GNU GPL v3.0

README.md - SURPRISE!  You're currently reading it!

check_update - This script is intended to be installed in root (or any other user's) crontab and executes at 0,15,30,45 minutes.  It first checks the central server's sync_ssh_keys directory looking for a file named 'update'.  This file can be generated however you see fit, I use touch.  Anyway, if this file exists, then it proceeds to download one or two files, depending on how the HAM variable is set.  If it is set to no, then only authorized_keys_per is retrieved.  If it is set to yes, then authorized_keys_ham is also retrieved.  Both files are then concatenated overwriting the existing authorized_keys in the current user's home directory.  There is no check for this, so if you value the contents of that file, BACK IT UP NOW!  You ARE reading this doc BEFORE running anything, right?????

clear_update - This script runs on the central server in root's (or any other user's) crontab at 5,20,35,50 minutes.  All it does is check for the existence of the file 'update' in the sync_ssh_keys directory and delete it.

The theory of operation is like this:  You edit the authorized_keys or authorized_keys_ham and add the desired PUBLIC ssh key.  Then you 'touch update' in the sync_ssh_keys directory.  When the remote script is triggered it will find the 'update' file and pull down the updated authorized keys file(s).

As always I'm open to suggestions or ideas, but for now this meets my needs.  As previously mentioned I do plan to clean things up and apply some code formatting to make this tool more like my others, but operationally, I do not intend to introduce any new features at this point as it does exactly what I needed it to do.

As always I'm posting this public as a learning tool for others.

73 de NF9K
Bill Atkinson (for you non-hams)

v1.5 - Significant changes here.  The code base was adopted to my function-based layout that I use in most of my other projects.  Not only does this clean the code up a bit, but it makes troubleshooting and testing easier by being able to test individual parts of the code.  Here is a list of the functional changes in this version:
 * The Ham option was moved to the command line.  So executing check_update with a parameter of 'yes' will trigger the ham option.  If you're using this option remember to update your cron jobs and add the parameter.
 * An upgrade routine was built in.  So if you 'touch upgrade' in the ~/sync_ssh_keys directory, the remote script will pull down a new copy.  Since this script was designed to run on many remote servers, I wanted an easy way to upgrade the scripts on each server.  The script currently pulls the two script files only.  It does not grab the README.md or the GPL license.
 * A sleep routine was added before any operation takes place.  The script will randomize a variable between 1-60 seconds and then sleep for that period before continuing.  This should stagger hits on the central server when multiple remotes are in play.
 * Added some null redirects to the SCP commands to clean up text output.
 * Added output to show progress
