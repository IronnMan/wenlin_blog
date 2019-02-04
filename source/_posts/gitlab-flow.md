title: GitLab 工作流
thumbnail: /img/page/default-blog-image.png
tags:
  - gitLab
categories: []
author: Sytse Sijbrandij
date: 2018-05-13 00:15:00
---
本博客文章内容的最新版本可以在 [GitLab 的文档](https://docs.gitlab.com/ee/workflow/gitlab_flow.html) 中找到。

使用 git 进行版本管理使得分支和合并比 SVN 等旧版本系统更容易，这允许各种各样的分支策略和工作流程。所有的这些，都是对 git 之前使用的方法的改进。但是许多组织最终的工作流程没有明确定义，过于复杂或不与问题追踪系统整合。因此，我们将 GitLab 工作流作为明确定义的一组最佳实践提出。它将 [功能驱动开发](https://en.wikipedia.org/wiki/Feature-driven_development) 和 [功能分支](https://martinfowler.com/bliki/FeatureBranch.html) 与问题跟踪结合在一起。

来自其他版本控制系统的组织经常发现难以开发有效的工作流程。本文将介绍 git 工作流程与问题跟踪系统集成的 GitLab 工作流。它提供了一种简单，透明和有效的方式来使用 git。

![Four Stages](https://about.gitlab.com/images/git_flow/four_stages.png)

当你使用 git 时，你与同事分享提交必须遵守三个步骤。大多数版本控制系统只有一个步骤，从工作副本提交到共享服务器。在 git 中，你可以将工作副本中的文件添加到临时区域。之后，你将它们提交到本地存储库。第三步是推送到共享远程存储库。习惯这三个步骤后，分支模式将成为挑战。

![Messy Flow](https://about.gitlab.com/images/git_flow/messy_flow.png)

由于许多刚接触 git 的新手没有约定如何使用它，它可能很快就会变得混乱。他们遇到的最大问题是许多长期运行的分支都包含了部分变化。人们很难弄清楚他们应该开发哪个分支或部署到生产环节。通常，对这个问题的反应是采用标准模式，例如 [git 工作流](http://nvie.com/posts/a-successful-git-branching-model/) 和 [GitHub 工作流](http://scottchacon.com/2011/08/31/github-flow.html)。我们认为还有改进的余地，并且会详细介绍一组我们称之为 GitLab 工作流的实践。


## Git 工作流及其问题

![git dash flow](https://about.gitlab.com/images/git_flow/gitdashflow.png)

Git 工作流是使用 git 分支的第一个建议之一，并且它受到了很多关注。它主张一个主分支和一个单独的开发分支，以及支持功能，发布和修补程序的分支。开发发生在开发分支上，转到发布分支并最终合并到主分支中。Git 工作流是一个明确的标准，但其复杂性引入了两个问题。第一个问题是开发人员必须使用开发分支而不是主分支，而主分支则保留用于发布到生产环境的代码。这是一个约定，称为你的默认主分支，并且大多数分支合并到此。由于大多数工具会自动使主分支成为默认分支，并默认显示该分支，所以不得不切换到另一个。git 工作流的第二个问题是修复程序和发行版分支引入的复杂性。对于一些组织而已，这些分支可能是一个好主意，但是对绝大多数组织来说却是过分的。现在大多数组织实行持续交付，这意味着你的默认分支可以部署。这意味着可以防止修补和释放分支，包括他们采用的所有仪式。这个仪式的一个例子就是发行分支的合并。虽然有专门的工具来决解这个问题，但它们需要文档并增加复杂性。开发人员经常犯一个错误，例如，更改只能合并到 master 中，而不是合并到 develop 分支中。这些错误的根本原因在于，对于大多数用例来说，git 工作流过于复杂。做发布并不意味着自己也在做修补程序。

## GitHub 工作流作为一个更简单的选择

![GitHub Flow](https://about.gitlab.com/images/git_flow/github_flow.png)

在对 git 工作流的反应中，更简单的选择是：[GitHub 工作流](http://scottchacon.com/2011/08/31/github-flow.html)。该流程仅具有功能分支和主分支。这是非常简单和干净的，许多组织已近取得了巨大的成功。Atlassian 推荐了 [类似的策略](https://www.atlassian.com/blog/archives/simple-git-workflow-simple)，尽管他们重新设置了功能分支。将所有内容合并到主分支并进行部署通常意味着你最大限度地减少「库存」中的代码量，这与精益和持续交付最佳实践一致。但是这一流程仍然留下了许多关于部署、环境、发布和问题集成的问题。借助 GitLab 工作流，我们为这些问题提供了更多指导。


## 生产分支与 GitLab 工作流

![Production Branch](https://about.gitlab.com/images/git_flow/production_branch.png)

GitHub 工作流假定您可以在每次合并功能分支时将其部署到生产环境。这对 SaaS 应用程序是可能的，但在很多情况下这是不可能的。一种情况是您无法控制确切的发布时刻，例如需要通过 AppStore 验证的 IOS 应用程序。另一个例子是当你有部署窗口时（操作团队满负荷工作时间从上午 10 点到下午 4 点），但你也可以在其他时间合并代码。在这些情况下，您可以使生产分支反映已部署的代码。您可以通过将主副本合并到生产分支来部署新版本。如果您需要知道生产环境中的代码，您只需要检出生产分支即可查看。作为版本控制系统中的合并提交，部署的大致时间很容易看到。如果您自动部署生产分支，这一次相当准确。如果您需要更准确的时间，则可以让您的部署脚步在每个部署中创建一个标签。此工作流可防止 git 工作流通用的释放，标记和合并开销。


## 环境分支与 GitLab 工作流

![Environment Branch](https://about.gitlab.com/images/git_flow/environment_branches.png)

将环境自动更新到主分支可能是一个好主意。只有在这种情况下，该环境的名称肯能与分支名称不同。假设您有一个临时环境，一个预生产环境和一个生产环境。在这种情况下，主分支在分段上部署。当有人想要部署到预生产时，他们会创建从主分支到预生产分支的合并请求。通过将预生产分支合并到生产分支来实现代码。这个工作流程只能在下游提交，确保所有环境都经过测试。如果您需要选择使用修补程序进行提交，通常在功能分支上开发它并用合并请求将其合并到主服务器中，请勿删除功能分支。如果主人是好的（应该是如果你练习 [持续交付](https://martinfowler.com/bliki/ContinuousDelivery.html)），然后将其合并到其他分支。如果这是不可能的，因为需要更多手动测试，则可以将功能分支的合并请求发送到下游分支。环境分支的「极端」版本正在为每个功能分支设置一个由 [Teatro](http://teatro.io) 完成的环境。


## 使用 GitLab 工作流发布分支

![Release Branch](https://about.gitlab.com/images/git_flow/release_branches.png)

只有在您需要向外界发布软件时，才需要使用发布分支。在这种情况下，每个分支包含次要版本（2-3-stable，2-4-stable 等）。stable 分支使用 master 作为起点，并尽可能晚地创建。通过尽可能晚地分支，您可以最大限度地减少将错误修正应用于多个分支的时间。在宣布发布分支后，发布分支中仅包含严重的错误修复。如果可能的话，这些错误修复程序首先合并到 master 中，然后挑选到发布分支中。这样你就不会忘记将它们挑选到 master 中并在后续版本中遇到相同的 bug。这被称为「 上游优先 」政策，也是 [Google](http://www.chromium.org/chromium-os/chromiumos-design-docs/upstream-first) 和 [Red Hat](https://www.redhat.com/en/blog/a-community-for-using-openstack-with-red-hat-rdo) 实施的政策。每次在发布分支中包含错误修复时，都会通过设置新标记来引发补丁版本（以符合 [语义版本控制](https://semver.org) ）。有些项目还有一个稳定的分支，指向与最新发布的分支相同的提交。在此流程中，拥有生产分支（或 git 工作流主分支）并不常见。


## 使用GitLab 工作流合并/拉取请求

![Mr Inline Comments](https://about.gitlab.com/images/git_flow/mr_inline_comments.png)

合并或拉取请求在 git 管理应用程序中创建，并要求指定人员合并两个分支。GitHub 和 Bitbucket 等工具选择名称拉取请求，因为第一个手动操作是拉取功能分支。GitLab 和 Gitorious 等工具选择名称合并请求，因为这是代理人请求的最终操作。在本文中，我们将它们称为合并请求。

如果你在功能分支上工作超过几个小时，最好与团队的其他成员分享中间结果。这可以通过创建合并请求而不将其分配给任何人来完成，而是提及描述中的人或评论（/cc @mark @susan）。这意味着它还没有准备好合并，但欢迎反馈。您的团队成员可以在一般情况下或在具体有行注释的特定行上对合并请求发表评论。合并请求用作代码审查工具，不需要 Gerrit 和 reviewboard 等单独的工具。如果审查发现缺点，任何人都可以提交并推动修复。通常，执行此操作的人是合并/拉取请求的创建者。当在分支上推送新提交时，合并/拉取请求中的差异会自动更新。

如果你对合并它感到满意，可以将其分配给最了解你正在更改的代码库的人，并提及你想要反馈的任何其他人。有更多反馈的空间，并且在指定的人对结果感到满意之后，分支被合并。如果分配的人感到不舒服，他们可以在不合并的情况下关闭合并请求。

在 GitLab 中，通常保存长期存在的分支（例如主分支），以便普通开发人员 [不能修改这些受保护的分支](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/permissions/permissions.md)。因此，如果要将其合并到受保护的分支中，请将其分配给具有主授权的人员。


## 问题与 GitLab 工作流

![Merge Request](https://about.gitlab.com/images/git_flow/merge_request.png)

GitLab 工作流是一种使代码和问题跟踪之间的关系更加透明的方法。

对代码的任何重大更改都应从描述目标的问题开始。Having a reason for every code change is important to inform everyone on the team and to help people keep the scope of a feature branch small. In GitLab each change to the codebase starts with an issue in the issue tracking system. If there is no issue yet it should be created first provided there is significant work involved (more than 1 hour). For many organizations this will be natural since the issue will have to be estimated for the sprint. Issue titles should describe the desired state of the system, e.g. "As an administrator I want to remove users without receiving an error" instead of "Admin can't remove users.".

当你准备好编码时，你可以从主分支开始解决问题。此分支的名称以问题编号开头，例如 「 15-require-a-password-to-change-it 」。

当你完成想要讨论代码时，你打开合并请求。这是一个讨论变更和审核代码的在线位置。创建分支是一种手动操作，因为你并不总是希望合并你推送的新分支，它可能是一个长期运行的环境或发布分支。如果你创建合并请求但未将其分配给任何人，则它是「 工作进程 」合并请求。这些用于讨论提议的实现，但尚未准备好包含在主分支中。

当作者认为代码准备就绪时，合并请求被分配给审阅者。审阅者在认为代码以准备好包含在主分支中时按下合并按钮。在这种情况下，代码将被合并，并生成合并提交，以便稍后可以轻松地看到此事件。即使可以在没有提交的情况下添加提交，合并请求也始终创建合并提交。这种合并政策在 git 中称为 「 no fast-forward 」。合并后，功能分支将被删除，因为不再需要它，在 GitLab 中，此删除是合并时的一个选项。

假设已合并分支但发生问题并重新打开问题。在这种情况下，重用相同的分支名称没有问题，因为它在合并分支时被删除了。每个问题最多只有一个分支。一个功能分支可能解决多个问题。

## 链接和关闭合并请求中的问题

![Close Issue Mr](https://about.gitlab.com/images/git_flow/close_issue_mr.png)

通过从提交信息（fixes #14，closes #67等）或合并请求描述中提及它们，可以实现与问题的链接。在 GitLab 中，这会在合并请求提到问题的问题中创建注释。合并请求显示链接的问题。将代码合并到默认分支后，这些问题将被关闭。

如果你只想在不关闭问题的情况下进行参考，你也可以提一下：「 Ducktyping is preferred. #12」。

如果你遇到跨多个存储库的问题，最好的办法是为每个存储库创建一个问题，并将所有问题链接到父问题。


## 压缩提交  rebase

![Rebase](https://about.gitlab.com/images/git_flow/rebase.png)

使用 git，你可以使用交互式 rebase （rebase -i）将多个提交压缩为一个并重新排序。如果你在开发期间对小更改进行了几次提交，并希望使用单个提交替换它们，或者如果你希望使顺序更符合逻辑，则此更能非常有用。但是，你永远不应该重命名已推送到远程服务器的提交。有人可以参考提交或精心挑选它们。重新绑定时，你更改提交的标识符（SHA1），这是令人困惑的。如果你这样做，将在多个标识符下知道相同的更改，这可能会导致很多混淆。如果人们已经审查过你的代码，那么如果你所有内容重新设置为一次提交，那么他们将很难仅查看你之后所做的改进。

鼓励人们经常提交并经常推送到远程存储库，以便其他人知道每个人都在做什么。这将导致每次更改的许多提交，这使得历史更难理解。但是，具有稳定标识符的优点超过了这个缺点。要理解上下文中的更改，可以始终查看合并提交，该合并提交将代码合并到主分支时将所有提交组合在一起。

After you merge multiple commits from a feature branch into the master branch this is harder to undo. If you would have squashed all the commits into one you could have just reverted this commit but as we indicated you should not rebase commits after they are pushed. Fortunately reverting a merge made some time ago can be done with git. This however, requires having specific merge commits for the commits your want to revert. If you revert a merge and you change your mind, revert the revert instead of merging again since git will not allow you to merge the code again otherwise.

Being able to revert a merge is a good reason always to create a merge commit when you merge manually with the `--no-ff` option. Git management software will always create a merge commit when you accept a merge request.


## Do not order commits with rebase

![Merge Commits](https://about.gitlab.com/images/git_flow/merge_commits.png)

With git you can also rebase your feature branch commits to order them after the commits on the master branch. This prevents creating a merge commit when merging master into your feature branch and creates a nice linear history. However, just like with squashing you should never rebase commits you have pushed to a remote server. This makes it impossible to rebase work in progress that you already shared with your team which is something we recommend. When using rebase to keep your feature branch updated you [need to resolve similar conflicts again and again](https://www.atlassian.com/blog/git/git-team-workflows-merge-or-rebase). You can reuse recorded resolutions (rerere) sometimes, but without rebasing you only have to solve the conflicts one time and you’re set. There has to be a better way to avoid many merge commits.

The way to prevent creating many merge commits is to not frequently merge master into the feature branch. We'll discuss the three reasons to merge in master: leveraging code, solving merge conflicts and long running branches. If you need to leverage some code that was introduced in master after you created the feature branch you can sometimes solve this by just cherry-picking a commit. If your feature branch has a merge conflict, creating a merge commit is a normal way of solving this. You should aim to prevent merge conflicts where they are likely to occur. One example is the CHANGELOG file where each significant change in the codebase is documented under a version header. Instead of everyone adding their change at the bottom of the list for the current version it is better to randomly insert it in the current list for that version. This it is likely that multiple feature branches that add to the CHANGELOG can be merged before a conflict occurs. The last reason for creating merge commits is having long lived branches that you want to keep up to date with the latest state of the project. Martin Fowler, in [his article about feature branches](https://martinfowler.com/bliki/FeatureBranch.html) talks about this Continuous Integration (CI). At GitLab we are guilty of confusing CI with branch testing. Quoting Martin Fowler: "I've heard people say they are doing CI because they are running builds, perhaps using a CI server, on every branch with every commit. That's continuous building, and a Good Thing, but there's no integration, so it's not CI.". The solution to prevent many merge commits is to keep your feature branches shortlived, the vast majority should take less than one day of work. If your feature branches commenly take more than a day of work, look into ways to create smaller units of work and/or use [feature toggles](https://martinfowler.com/bliki/FeatureToggle.html). As for the long running branches that take more than one day there are two strategies. In a CI strategy you can merge in master at the start of the day to prevent painful merges at a later time. In a synchronization point strategy you only merge in from well defined points in time, for example a tagged release. This strategy is [advocated by Linus Torvalds](https://www.mail-archive.com/dri-devel@lists.sourceforge.net/msg39091.html) because the state of the code at these points is better known.

In conclusion, we can say that you should try to prevent merge commits, but not eliminate them. Your codebase should be clean but your history should represent what actually happened. Developing software happen in small messy steps and it is OK to have your history reflect this. You can use tools to view the network graphs of commits and understand the messy history that created your code. If you rebase code the history is incorrect, and there is no way for tools to remedy this because they can't deal with changing commit identifiers.


## Voting on merge requests

![Voting Slider](https://about.gitlab.com/images/git_flow/voting_slider.png)

It is common to voice approval or disapproval by using +1 or -1 emoticons. In GitLab the +1 and -1 are aggregated and shown at the top of the merge request. As a rule of thumb anything that doesn't have two times more +1's than -1's is suspect and should not be merged yet.


## Pushing and removing branches

![Remove checkbox](https://about.gitlab.com/images/git_flow/remove_checkbox.png)

We recommend that people push their feature branches frequently, even when they are not ready for review yet. By doing this you prevent team members from accidentally starting to work on the same issue. Of course this situation should already be prevented by assigning someone to the issue in the issue tracking software. However sometimes one of the two parties forgets to assign someone in the issue tracking software. After a branch is merged it should be removed from the source control software. In GitLab and similar systems this is an option when merging. This ensures that the branch overview in the repository management software shows only work in progress. This also ensures that when someone reopens the issue a new branch with the same name can be used without problem. When you reopen an issue you need to create a new merge request.


## Committing often and with the right message

![Good Commit](https://about.gitlab.com/images/git_flow/good_commit.png)

We recommend to commit early and often. Each time you have a functioning set of tests and code a commit can be made. The advantage is that when an extension or refactor goes wrong it is easy to revert to a working version. This is quite a change for programmers that used SVN before, they used to commit when their work was ready to share. The trick is to use the merge/pull request with multiple commits when your work is ready to share. The commit message should reflect your intention, not the contents of the commit. The contents of the commit can be easily seen anyway, the question is why you did it. An example of a good commit message is: "Combine templates to dry up the user views.". Some words that are bad commit messages because they don't contain munch information are: change, improve and refactor. The word fix or fixes is also a red flag, unless it comes after the commit sentence and references an issue number. To see more information about the formatting of commit messages please see this great [blog post by Tim Pope](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html).


## Testing before merging

![CI Mr](https://about.gitlab.com/images/git_flow/ci_mr.png)

In old workflows the Continuous Integration (CI) server commonly ran tests on the master branch only. Developers had to ensure their code did not break the master branch. When using GitLab flow developers create their branches from this master branch so it is essential it is green. Therefore each merge request must be tested before it is accepted. CI software like Travis and GitLab CI show the build results right in the merge request itself to make this easy. One drawback is that they are testing the feature branch itself and not the merged result. What one can do to improve this is to test the merged result itself. The problem is that the merge result changes every time something is merged into master. Retesting on every commit to master is computationally expensive and means you are more frequently waiting for test results. If there are no merge conflicts and the feature branches are short lived the risk is acceptable. If there are merge conflicts you merge the master branch into the feature branch and the CI server will rerun the tests. If you have long lived feature branches that last for more than a few days you should make your issues smaller.


## Merging in other code

![Git Pull](https://about.gitlab.com/images/git_flow/git_pull.png)

When initiating a feature branch, always start with an up to date master to branch off from. If you know beforehand that your work absolutely depends on another branch you can also branch from there. If you need to merge in another branch after starting explain the reason in the merge commit. If you have not pushed your commits to a shared location yet you can also rebase on master or another feature branch. Do not merge in upstream if your code will work and merge cleanly without doing so, Linus even says that [you should never merge in upstream at random points, only at major releases](https://lwn.net/Articles/328438/). Merging only when needed prevents creating merge commits in your feature branch that later end up littering the master history.


> 👉[原文地址](https://about.gitlab.com/2014/09/29/gitlab-flow/)