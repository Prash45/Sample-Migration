# Sample-Migration
Migration test

Overview

This is the process I have found to be very helpful when migrating projects from their SVN repository over to Git. When migrating from one version control system to another it's important to retain as much information about the development history as possible (i.e., commit messages, tags, branches, etc.) and avoid the easy and tempting method of just downloading the project from a server and throw it into the new version control (you should only consider that if the project has never been in a version control system before).

Key concepts:

tools for migrating svn repositories to git
how to migrate standardized svn repositories
how to migrate non-standardized svn repositories
best practices for your now un-maintained SVN repository
setup git-flow for your new git repository
tweaks to Capistrano deployment recipes
Tools for Migrating SVN to Git

If you've Googled for how to migrate from svn to git then you've probably come across a lot of results and stack overflow questions/answers on how to do it and the common theme out there is to use Git's internal svn extension (git-svn) to do so. You've probably also seen references to the ruby gem svn2git that wraps git-svn and offers easy to use options to handle almost any scenario or state of discord your SVN repo may be in. I've utilized both approaches in the past and I must say that svn2git is way easier than the manual process of doing it all through git-svn. Github also highly recommends the use of svn2git to migrate projects over to git so just do it already.

Install svn2git

First, you'll need to make sure you have git, git-svn and ruby installed on your system. If necessary, please refer to the installation instructions provided by svn2git's README file for more details.

  $ gem install svn2git
How to migrate standardized SVN repositories

"Standardized" in this case means that the root directory contains the tags, trunk, and branches directories like this example:

path/to/my/project/repo/
  |- branches
    |- debug_checkout
    |- new_homepage
  |- tags
    |- v1.0.0
    |- v1.1.0
    |- v1.1.3
  |- trunk
    |- project files located here...
The following code example is taken from svn2git's README

  $ svn2git http://svn.example.com/path/to/repo
How to migrate non-standardized svn repositories

Again, please refer to svn2git's README for scenarios, but the combination I've had to use the most is where:

The svn repo is NOT in standard layout and has no trunk, branches, or tags at the root level of the repo. Instead the root level of the repo is equivalent to the trunk and there are no tags or branches.

$ svn2git http://svn.example.com/path/to/repo --rootistrunk
Typically this requires a user with permission to access the project so it may be necessary to append the --usernmame <<user_with_perms>> bit to the above example like so:

  $ svn2git http://svn.example.com/path/to/repo/trunk/htdocs --rootistrunk --username <<user_with_perms>> --authors <<path_to_your_authors.txt_file>>
Setup git-flow for your new git repository

If you're unfamiliar with the git branching workflow, I highly suggest you read the quick overview posted by Jeff Kreeftmeijer and read Vincent Driessen's blog post A successful Git branching model and finally install the git-flow extension ( Github repo ).

If you completed the svn2git migration from above, then you will have a fresh git repository and will probably be in the master branch by default. To setup the project to use the git-flow branching model we'll need to create a branch that will be used for bleeding-edge development (features should still be considered stable but aren't quite ready for production environment). If you follow the git-flow best practices you'll probably name this bleedging-edge branch develop and reserve the master branch for production-ready code. But before we setup the git-flow workflow for our project, let's make sure a proper .gitignore file exists for the project.

Add a .gitignore file to your project

IMPORTANT NOTE: Don't just assume you should check everything into git! That's a really bad habit and can cause a lot of headaches when deploying your code to live environments.

If you're working with Magento, I'd suggest you use the .gitignore file from my AAI_Magento.gitignore gist as a starting point; otherwise take a look at Github's gitignore repository for common .gitignore files for common frameworks or applications.

Example: Download the example AAI_Magento.gitignore file and compare it to your existing .gitignore file (if present) for any things you may want to keep or change. If you do not have an existing .gitignore file you can simply rename the example AAI_Magento.gitignore to .gitignore.

wget https://gist.github.com/grafikchaos/5829233/raw/cd47916e5f0d30f6cf95bec67f529cbbcf3fb9fe/AAI_Magento.gitignore
mv AAI_Magento.gitignore .gitignore
Setup the project for git-flow

The following series of commands will:

create the develop branch based off the master branch
add the remote origin (Github in this case)
push the local develop branch up to the remote repository
runs the git-flow initialization script (you can probably just leave the default values for every question)
  $ git checkout -b develop
  $ git remote add origin git@github.com:<GITHUB_ACCOUNT>/<REPOSITORY_NAME>.git
  $ git push -u origin develop
  $ git flow init
What to do with your old SVN repo?

Best practices for your now un-maintained SVN repository

An orphaned repository is a headache just waiting to happen, especially if you work in a team or group environment. Either somebody will have a legacy checkout of the application and try to upload legacy files into your fresh and clean application or somebody will be confused as to what repo they should checkout and somebody WILL clone the SVN version instead of Git and waste their time and effort on a defunkt repo. You can avoid these headaches by wiping the SVN repo and committing a new README.txt to direct all those who are lost to the new repository location.

The most efficient way I've found of wiping an SVN repo comes from this Stack Overflow answer. Essentially the process goes as follows:

  $ svn checkout --depth immediates http://myrepo.com/svn/myrepo myworking_copy
  $ cd myworking_copy
  $ svn rm *
  $ svn rm .htaccess .*
  $ svn ci -m "Deleting all files after migrating to github"
  $ touch README.txt
  $ echo "Project has moved to Github! Check it out at https://github.com/<GITHUB_ACCOUNT>/<REPO_NAME>" >> README.txt
  $ svn add README.txt
  $ svn ci -m "adds README to point lost souls to new repo location"
Tweaks to Capistrano recipes

If your project was already set up to use capistrano or our wonderful capistrano-ash recipes then you will need to do the following:

Setup deploy keys as necessary
You can refer to this capistrano-ash wiki page for instructions on how to create a deploy key on the server.
For a better understanding of how and when to use deploy keys, read the Managing Deploy Keys article from Github's Help
SSH into the servers that have been deployed to

Go to the shared directory and either move or copy the old cached copy directory (typically named cached-copy or cached_copy but newer versions of capistrano may follow different naming patterns).
Example code to run: cd /path/to/my/project/<environment>/shared && mv cached-copy cached-copy_svn
From your local machine's project directory run cap <environment> deploy:setup to tell capistrano to create a new cached-copy (or similarly-named directory) on your server(s)

NOTE: If you are hosting on a shared environment you may need to go back into the server and change directory permissions on the deploy_to directory (typically a chmod 755 <directory_name> does the trick)
optional: run cap <environment> deploy:check to see if everything is setup as it's supposed to be. This isn't required, but it's a good habit to get into.
Deploy the application with a normal cap <environment> deploy
