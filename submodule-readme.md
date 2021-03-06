# Working with submodules

Some key concepts:

* At any given time your main repository needs to point to a *specific commit* in your submodule. On your computer, the files shown in the submodule folder will match the contents at that specific commit.

* When you update your submodules, by default your main repository will shift to point to the most recent commit on the *master branch* of the submodule. Main repositories can be configured to point to the most recent commit on another branch, if desired. Check the `.gitmodules` file to see if a branch is specified. If no branch is specified, updating your submodule will change the submodule contents to the most recent commit on the master branch.

* Pointing to a specific commit is not the same as *tracking a branch*. In order to commit and push changes in your submodule, you will need to ensure that your main repository is tracking a specific branch of the submodule.

* Confusingly, updating your submodule will change your main repository to not track any branch of your submodule. If you are not tracking, you can lose your data if you make local changes to the submodule and then update your repository again (which instantly moves the content to the newest commit, disregarding anything local, because it wasn't tracking).

* Seeing files in your submodule directory is **not** the same as tracking them. At all times we need to make sure (A) our submodule points to the most recent commit on the desired branch, and (B) we are tracking that same branch. Luckily, these are both easy tasks to accomplish -- we just need to remember to do them each time!


## Quick links

If you don't want to follow the instructions below every time, skip ahead to [Setting up custom git commands for dealing with submodules](#setting-up-custom-git-commands-for-dealing-with-submodules). After setting up these commands, `git-sclone`, `git-spull`, and `git-spush` will clone, pull, and push repositories while accounting for submodules.

* [Cloning a repository with submodules](#cloning-the-repository-for-the-first-time)
* [Pulling changes](#pulling-changes-to-git-repositories-with-submodules)
* [Making changes to submodules](#making-changes-to-submodules-within-the-main-repository)
* [Pushing changes](#pushing-changes-to-git-repositories-with-submodules)
* [Switching branches](#switching-branches-when-working-with-submodules)

## Cloning a repository with submodules for the first time

To properly clone a git repository with submodules, run the following command for the main repository:

```
git clone --recurse-submodules REPOSITORY-URL
```

If you have already cloned the repository with the standard `git clone REPOSITORY-URL`, you will see the directories that contain the submodules, but they will be empty. In that case, you need to run the following two commands afterwards to fetch all the data from these submodules. **Don't run these commands if you properly cloned your repository with the --recurse-submodules flag above.**

```
git submodule init
git submodule update
```

Afer cloning your repository, run the following command to ensure that you are pointing to the most recent version of each submodule *in the specified branch for this repository*. You can check the branch that this repository points to by looking inside `.gitmodules`. If no branch is specified, this repository defaults to master.

```
git submodule update --remote --recursive
```

(This will update all of the submodules in your main repository. If there are many, consider adding `SUBMODULE-NAME` to the end of the command to update just one at a time.)

Since we just updated which commit in the submodule the main repository is pointing to, we need to commit these changes:

```
git add SUBMODULE-NAME
git commit -m 'update submodule version to most recent commit'
```

We also need to ensure we are tracking the desired branch so that any local changes don't get lost. This step isn't strictly necessary if you do not plan to update the submodule, but it's good practice to get in the habit of. The command below will ensure each submodule tracks the branch specified in the `.gitmodules` file.

```
git submodule foreach -q --recursive 'git checkout $(git config -f $toplevel/.gitmodules submodule.$name.branch || echo master)'
```

## Pulling changes to git repositories with submodules

Each time you want to pull new changes, including new changes to the submodules, do the following. Remember, the basic steps to updating submodules are:

1. Pull
2. Commit
3. Track

```
git pull
git submodule update --remote --recursive

git add SUBMODULE NAME
git commit -m 'git commit -m 'update submodule version to most recent commit'

git submodule foreach -q --recursive 'git checkout $(git config -f $toplevel/.gitmodules submodule.$name.branch || echo master)'
```

Note: a previous version of this document suggested using `git pull --recurse-submodules` instead of `git pull` followed by `git submodule update --remote --recursive`. This will still work as expected if your main repository points to the master branch of the submodule. However, when this is not the case, `git pull --recurse-submodules` will not update the submodule to the correct commit. Therefore, this has been updated to the current version, which will work regardless of the designated submodule branch.

#### Updating your submodule to the most recent commit on another branch

There is a way to update the submodule to the most recent commit on a branch other than the one specified in the `.gitmodules` file of your main repository (remember: if no branch is specified, this repository will by default update to the most recent commit of the master branch of the submodule).

One way to do this is by updating the `.gitmodules` file. **However, the `.gitmodules` file is typically pushed and pulled, so updating the branch here will modify it for everyone.** If you just want to temporarily change the branch of your submodule, it is safer to update your local `.git/config` file. *Just remember to change it back when you are done*, as the content of this file will override what is in the `.gitmodules` file.

Update your `.git/config` file to point to a different submodule branch like this:

```
git config -f .git/config submodule.SUBMODULE-NAME.branch SUBMODULE-BRANCH-NAME
```

After doing this, make sure to run `git submodule update --remote --recursive` to pull updates from the most recent commit on this desired branch.

## Making changes to submodules within the main repository

*If you are not very familiar with submodules, you can reduce possible confusion by only updating the submodule from within the submodule repository itself, and not from within the main project repository (i.e., don't make changes to COVIDScenarioPipeline from within another project folder -- navigate to a separate directory containing only the COVIDScenarioPipeline repository.)*

Before making any changes within the submodule directory, you need to check out a specific branch of that submodule. As described above, this is very important so that you don't accidently lose local changes.

If you correctly followed the steps to pull changes and update your submodule in the section above, this step should be redudant. But let's do it just to be sure. This also shows you how to track a different branch, should you ever want to work on the non-default branch of your submodule.

```
cd SUBMODULE-NAME
git checkout SUBMODULE-BRANCH-NAME
```

Now you can make local changes and commit them as usual. Make sure to read the section below on how to properly push changes when you have been editing the submodule.

## Pushing changes to git repositories with submodules

Use the following command to push changes, including all committed submodule changes:

```
git push --recurse-submodules=on-demand
```

## Switching branches when working with submodules

When working with submodules, switch branches in the main repository like this to avoid any issues:

```
git checkout --recurse-submdoules BRANCH-NAME
```

To avoid potential issues, make sure to commit and push all changes on a branch before switching. After switching, make sure to update this branch to point to the most recent commit on the submodule branch specified in this branch of the main repository, commit this change, and set up tracking of the submodule directory.

## Setting up custom git commands for dealing with submodules

Following these steps will create custom commands that you can use to clone, pull, and push repositories without worrying about submodules. However, remember that **`clone` and `pull` may create changes that need to be committed** -- don't forget to commit these changes (typically with `git commit -m 'update submodule version to most recent commit'`) after running!

#### Linux or Mac OSX

1. Copy the `gitcommands.sh` file in this repository to your machine
2. Open your `.bash_profile` file and add the following: `source PATH-TO/gitcommands.sh`
3. Close your terminal window

After restarting your terminal, the commands `git-sclone`, `git-spull`, and `git-spush` should work just as you expect, but they will account for submodules in your repositories. You run them like this:

```
git-sclone URL
git-spull
git-spush
```

#### Windows

**I don't have a windows computer so this part has not been tested yet.**

1. Copy the `gitcommands.bat` file in this repository to C:/Windows/System32 on your machine
2. Close your Command Prompt

After restarting your Command Prompt, the commands `git-sclone`, `git-spull`, and `git-spush` should work just as you expect, but they will account for submodules in your repositories. You run them like this:

```
git-sclone URL
git-spull
git-spush
```