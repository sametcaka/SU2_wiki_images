The repository for SU2 is being hosted on here on GitHub. As you are likely aware, GitHub is simply an online project hosting service with a very useful web interface and additional tools to aid code development with Git as its backbone. Git is a version control system (VCS) which is similar to SVN, Mercurial, etc., and it helps organize the development of code over time by tracking changes. 

To get started, you need to create a personal user account on GitHub (free) and follow the [basic setup instructions](https://help.github.com/articles/set-up-git). These instructions include how to get Git installed on your local machine. To sync up your local settings with GitHub, change the user.email and user.name variables for your local git configuration with
```
git config --global user.email "your_email@domain.com" 
git config --global user.name "Your Name"
```
Note that the email address should be the one associated with your GitHub account.

SU2 is an open-source repository that can be found as part of the [**su2code** organization on GitHub](https://github.com/su2code). The web interface is useful for viewing the recent commits to the code, changes to the code over time, documentation and tutorials, or for creating and viewing branches, for instance. To contribute to SU2 directly, an administrator for the project must add you as a member of the developer team with push and pull privileges. However, we encourage all developers to fork the repository and to submit pull requests with their interesting new features that they would like included in the code. We regularly review pull requests as a development team.

Most of the day-to-day development of the code will be done on your local machine at the command line using Git. After setting up Git and gaining access to the SU2 repository, create a local copy of the entire repository on your machine by cloning the version that is hosted on GitHub:
```
git clone https://github.com/su2code/SU2.git
```
After cloning, you should have a new SU2/ folder in your current working directory. Move into SU2/ to see the project files and to start working with the code through Git. You can see the most recent changes by typing
```
git log
```

## Typical Workflow with Git

Now that you have a local copy of SU2 from the GitHub repository, you can begin to make changes to the codebase. This section gives an example of the typical workflow for making changes to the code, committing them locally, and then pushing your changes to the remote GitHub repository. The basic steps are as follows:
 
1. Make changes to the existing files (using your favorite text editor or integrated development environment, IDE) or add local files or folders to be tracked and compared against the global repo files.```git add file1.cpp file2.cpp```
2. Optionally check that your changes have been registered and/or the files that you want have been added added
```
git status 
```
3. Commit the changes to your local repository (not the global repository on GitHub) and leave a short descriptive message about your change.
```
git commit -am "Added some files."
```
4. Merge local and global repositories.
This command will attempt to merge your version of the code with the global
version. Near the end of the merge process, git will tell you if everything has
been merged successfully. If there are conflicts, it will tell you the files
that contain the conflicts. You must then navigate to these files, open them,
and resolve the issues. The conflicting regions of code are delimited with
chevrons like this \texttt{>>>>>>} and \texttt{<<<<<<}. Note that, if you experience and resolve conflicts, you will need to perform an additional local commit of the resolved files afterwards.
```
git pull origin master
```
5. Push the final version of the code to the global repository on GitHub (the remote repository is named `origin' by default). The changes you have made will now be available to all, and they will also be almost immediately reflected on the SU2 page on GitHub.
```
git push origin master 
```

## Branching in Git

The ease of code branching is a major feature in Git. Branches are parallel versions of the code that allow for decentralized development of particular features or fixes. In this manner, an individual or team can easily switch between developing different features in the code and merge them into the master version when ready (this can also help avoid conflicts). In fact, while not mentioned above, the master version of the code is simply a branch like any other. To see the branches in your local repository, type
\begin{verbatim}
git branch
\end{verbatim}
The branch name with an asterisk is the current working branch. One can add branches locally or globally to the remote repository on GitHub that can be shared by all. Assume that a new branch named `feature\_new' has been created in the remote repository, either through the command line or through the GitHub web interface, and you would like to work on this feature. A typical workflow in this scenario might be:

\begin{enumerate}

\item Clone a fresh copy of SU2 or update your current version with the latest changes on the remote repository.
\begin{verbatim}
git pull
\end{verbatim}
If you would just like to make your local repository aware of changes in the remote (such as the addition of the `feature\_new' branch), but don't want any changes to local files, you can use
\begin{verbatim}
git fetch
\end{verbatim}

\item Create a local copy of the branch that is linked to the version on the remote repository,
\begin{verbatim}
git checkout -b feature_new origin/feature_new
\end{verbatim}
This command creates the local branch and switches your local working copy to the `feature\_new' branch. Note that you can not have any local edits or changes when switching between branches, so you should make a local commit of your changes (as described above), stash them, or revert all local changes in the repository by entering the following command in the root directory of the source distribution (i.e., SU2/),
\begin{verbatim}
git checkout -- .
\end{verbatim}

\item Check that you are now in the `feature\_new' branch with
\begin{verbatim}
git branch
\end{verbatim}
You should notice that the local copies of your files have seamlessly switched to their state under the `feature\_new' branch.

\item Make some changes to the code and commit your changes to your local copy of the `feature\_new' branch as usual
\begin{verbatim}
git commit -am "Updates."
\end{verbatim}

\item Merge local and remote versions of the branch and fix any conflicts if necessary,
\begin{verbatim}
git pull origin feature_new
\end{verbatim}

\item Push the final version of the code to the remote branch on GitHub to make it available to all,
\begin{verbatim}
git push origin feature_new 
\end{verbatim}

\end{enumerate}

It is often the case that you would like to merge your branches back into the master branch after completing work on your new feature or bug fix. When developing features that may take an extended amount of time, it is a good idea to update your branch frequently with the recent changes in the master. This will make it much easier to merge the branches eventually and will help avoid conflicts and headaches when the time comes. Let's assume you are about to work on the `feature\_new' branch again, but would like to update it with the most recent work in the master branch. You could do the following (there are multiple ways to push/pull things between branches):

\begin{enumerate}

\item Move back over into your local copy of the master branch
\begin{verbatim}
git checkout master
\end{verbatim}

\item Check that you are back in the master branch with
\begin{verbatim}
git branch
\end{verbatim}

\item Pull the latest and greatest from the remote master branch on GitHub. Your local copy of the master branch now has all recent changes that can be shared with other local branches.
\begin{verbatim}
git pull origin master
\end{verbatim}

\item Switch back over to your local copy of the `feature\_new' branch
\begin{verbatim}
git checkout feature_new
\end{verbatim}

\item Merge the local version of the master branch that you just updated into your local version of `feature\_new' and fix any conflicts if necessary.
\begin{verbatim}
git merge master
\end{verbatim}

\item Make code changes, merge with the remote `feature\_new' branch, and push to the remote `feature\_new' branch like usual
\begin{verbatim}
git commit -am "More updates."
git pull origin feature_new
git push origin feature_new 
\end{verbatim}

\end{enumerate}

Note that this process could be more direct by pulling from the remote master straight from within your local copy of the `feature\_new' branch. Again, there are multiple ways of updating and merging that involve pushing and pulling between different branches that may be local or remote.

Lastly, once you are finished developing your new feature and have merged your work from the `feature\_new' branch into the master, you can delete the branch from the remote repository with
\begin{verbatim}
git push origin :feature_new
\end{verbatim}