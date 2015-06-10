Code review is a process where changes to the SU2 repository are examined by other developers before being merged into the development branch

# Purpose
The goal of code review is to maintain correct code in a consistent style. Humans make mistakes, and code review is in place to catch mistakes before they become part of the main code suite. Good code is not just correct, but is also legible, and what is clear to the code author may not be clear to a reader of the code. Code review ensures that the code is understandable by at least two developers

# Philosophy
All developers and users (internal and external) are encouraged to participate in the code review process. The SU2 suite is a computational environment we are all working to maintain. When performing a code review, you should be asking yourself "Is this code that I want in our environment". A single developer may have written the bulk of the pull request, but once a change has been incorporated the whole community is in charge of future changes (maintenance, debugging, etc.). Questions you should ask yourself are

1. Is this a desirable change to the code base? 
- Does it make code more legible? 
- Add new features? 
- Fix a bug?

2. Is the change implemented in the correct way?
- Does it interact minimally with the rest of the code?
- Does it have the correct algorithmic complexity?
- Is it located in the place? (file, etc.)

3. Is the code legible?
- Is the code mostly legible on its own without documentation?
- Are the variable names concise and accurate?
- Is there documentation where necessary, and it is correct?

4. Does the code follow established conventions?
- Does it match the SU2 code style?
- Do the variable names follow the same patterns as in other parts of the code?

# Good code changes
The above list is a long list of questions. A large change to the code will be much harder to review than a small change. As such, good pull requests will contain a minimal set of changes to be useful. Pull requests should "do one thing". In some cases, "doing one thing" may be a large change to the code, such as adding a new flow solver. In most cases, changes can be done in small increments that can each individually be reviewed and evaluated. Pull requests should not be of the form "Add a new config option and fix a bug in interation_structure.cpp". These should be two separate pull requests.

# The Code Review Process
Github provides an easy interface for performing code reviews. On the main "conversation" tab, there is a dialog box for making high-level suggestions about the code. These may be things like "This change belong in file X instead" or "You also need to change where this occurs in file Y". Comments are also possible on specific code issues, on the "Files changed" tab, there are little "+" signs at each line changed. These are comments to that specific section of the code, such as rewording documentation, renaming variables, or fixing whitespace. 

In response to these comments, the pull requester should make changes to the code (or engage in dialogue on disagreements). Github does not alert when changes have been uploaded, and so the requester should use the main dialogue box to alert that changes have been made. Traditionally this is done with the characters "PTAL", which stand for "Please take a(nother) look".

When a reviewer is happy with the proposed changes to the code, the reviewer says "LGTM", standing for "looks good to me". In general, the changes to the code should be finalized before a LGTM is given, though if there are only very minor outstanding issues an LGTM can be given along with the changes. For example, if the only outstanding issue with the PR is that a word has been misspelled, a reviewer may make an inline comment about the misspelling, and in the main dialogue say "LGTM with the comment fix".

Pull requests should always wait for an explicit "LGTM" before merging. If a reviewer seems happy but has not given one, feel free to ask. The explicit LGTM limits miscommunication between reviews and developers. "I like the way this looks" is NOT an okay to merge, it may be that the reviewer has minor fixes they'd like to suggest. One exception to this rule is when fixing breaking changes. If SU2 no longer compiles (for example) and your change allows it to compile, it is okay to merge to make that fix. After it has been merged, another developer should still review the changes and give a post-facto LGTM, or request a different fix to the problem.

All developers are encouraged to participate in the code review process, as the code is for everyone. However, there will typically be a specific set of developers who are expert in the section of code that is being modified. Generally, an LGTM should be gotten from at least one of these developers before merging. Users can be requested using "@username", for example, "PTAL @su2luvr". This sends that user an email about the pull request. Similarly, this can be used to request the opinions of other developers. While this can feel burdensome, it is in place to maintain good, correct code. Please use good judgement -- if the change is a spelling fix in a comment, it is not necessary to solicit the opinion of the entire development team.

Once the proper set of "LGTM"s has been received, the change can be merged. If the pull-requester has commit access, it is tradition to let them make the merge. If the requester does not, then the main/final reviewer can submit the merge. If the pull-request came from an internal branch, the branch should be deleted on conclusion if it is no longer useful.