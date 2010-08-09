Piston Exporter
===============

Developed by Sam Minnée while at SilverStripe.

[Piston](http://piston.rubyforge.org) is an excellent tool for managing your own project-specific branches of third party code.  I use it myself for managing my SilverStripe projects, as it lets me make small improvements and bugfixes to the SilverStripe code while still giving me the opportunity to upgrade.

However, what if I want to get those bugfixes back into the core project?  Piston doesn't allow for this out of the box.

Enter `piston-export`.

Installation
------------

Clone this repository and symlink piston-export to appear somewhere in your path.

How to use it
-------------

Let's imagine that you have a project with a piece of code imported using piston.  Here I am using SilverStripe's sapphire module as an example, because that's what I'm familiar with.

    cd ~/Sites/myproject
    piston import http://svn.silverstripe.com/open/modules/branches/sapphire/2.4 sapphire

You've made some changes in the sapphire directory, run `piston update` a few times maybe, and now you want to submit those changes back to the core module.

 * Create a git clone of the original project repository.   If that is a subversion repository, then you should use `git svn` to do this.

        cd ~/Projects/gitsvn
        git svn clone http://svn.silverstripe.com/open/modules/branches/sapphire/2.4 sapphire
        
 * If you had created this repository previously, you might want to make sure it's nice & clean
 
        cd ~/Projects/gitsvn/sapphire
        git checkout -f master
        git clean -fd
        git am --abort

 * Go to your project and run piston export, passing the piston imported directory and the newly created git clone of the original source, and optionally a branchname.
 
        cd ~/Sites/myproject
        piston-export sapphire ~/Projects/gitsvn/sapphire stuff-from-myproject

 * Assuming there were no conflicts your repository will contain a number of changes in a stuff-from-myproject branch.  I'd suggest that you use `git rebase --interactive` tidy these up before committing.
 
        cd ~/Projects/gitsvn/sapphire
        git checkout stuff-from-myproject
        git rebase --interactive master


 * Here are some of the things that you can tidy up with interactive rebasing:
   * Alter the commit messages to remove any references to your project and make them appropriate for the main repository.
   * Reorder the commits so that the ones that can be merged straight away come first, and the ones that still need review come second.  This will keep the commit history more linear if you are only going to `git svn dcommit` a portion of the commits to begin with.
   * Remove any commits that are inappropriate, for example a change that is subsequently reverted, or a project-specific change that should never make its way into the main repository.
   * Squash multiple commits for a single feature together into a single commit.  This will keep the main repository's history more concise.

 * Alternatively, you could cherry-pick some of the changes into a separate branch, letting you commit some now and leave some for later.  I'm personally a fan of using GitX for this purpose.
 
 * Once you're ready to commit them back, you can submit them to the core project in whatever way suits you best.
 
   * If you're on the core team for that project you could commit them directly.
   * You could use `git format-patch` to create a series of patches to send to the core team.
   * You could push them to a GitHub fork of the project and file a pull request.

**Note:** `piston-export` will add an `exported_to` entry to .piston.yml, which means that piston-export can be called multiple times.

### What if there were conflicts?

If any of the patches weren't able to be applied, you will need to continue the `git am` execution yourself.  `git am` is the tool that is used to apply the patches.  First, go to the directory where you were applying files.

    cd ~/Projects/gitsvn/sapphire
    
The pieces of the patch that could be applied will be included in the unstaged changes.  The other pieces will be included in .rej files.  Review the .rej files and manually apply them.  Then git add the files that should be part of this patch:

    git add <altered-files>
    git am --resolved
    
Alternatively, you may decide that this patch shouldn't be applied:

    git am --skip

How does it work?
-----------------

`piston-export` uses `git log` to identify all of the changes made to the subdirectory that was imported with piston, and then ignores any commits that altered `.piston.yml`, because these are going to be `piston update` and `piston-export` calls.

It uses `git format-patch` to create a set of patch files for all applicable changes in a temporary directory (a subdirectory of the destination repository called `piston-export-yyymmdd`).

Finally, it uses `git am` to apply those patches to a new branch in the destination repository.  If this fails it will choke and let you finish off the job yourself.

It will also add an `exported_to` entry to `.piston.yml`, listing the current HEAD SHA.  Subsequent calls will only create patches of commits created after this SHA.

Limitations
-----------

 * Currently, piston-export only works if you have `piston import`ed into a git project.
 * I'm not sure what happens if you run piston-export and then rebase, meaning that `exported_to` will list an invalid SHA.

Licence
-------

Copyright 2010 SilverStripe Limited. All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are
permitted provided that the following conditions are met:

   1. Redistributions of source code must retain the above copyright notice, this list of
      conditions and the following disclaimer.

   2. Redistributions in binary form must reproduce the above copyright notice, this list
      of conditions and the following disclaimer in the documentation and/or other materials
      provided with the distribution.

THIS SOFTWARE IS PROVIDED BY <COPYRIGHT HOLDER> ``AS IS'' AND ANY EXPRESS OR IMPLIED
WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND
FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> OR
CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

The views and conclusions contained in the software and documentation are those of the
authors and should not be interpreted as representing official policies, either expressed
or implied, of SilverStripe Limited.