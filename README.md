# git-issue-ops
---
#### How it Works!

Repository [np-reddy/git-issue-ops](https://github.com/np-reddy/git-issue-ops) is used for defining the necessary objects and workflows to create repository in Artifactory.

#### Prerequisites:
- [Issue template](https://github.com/np-reddy/git-issue-ops/blob/main/.github/ISSUE_TEMPLATE/artifactory-repo-creation-request.md) to capture details of github repo from which users needs to be synced to artifactory, artifactory repo type and package type.
- [Artifactory Pro or Enterprise](http://34.133.106.71/ui/login/) which is accessible from internet or from the actions runner (if using self-hosted).
- [Setup JFrog CLI](https://github.com/jfrog/setup-jfrog-cli#creating-the-configuration-on-your-local-machine) on your local machine, export the token/key and save it as secret in your GH repo.
- Generate and save github token with organization owner access.
- Create and modify [Artifactory object creation action workflow](https://github.com/np-reddy/git-issue-ops/blob/main/.github/workflows/arti-object-creation.yml) according to your environment.
- Create and modify [Artifactory templates](https://github.com/np-reddy/git-issue-ops/tree/main/artifactory-templates) that suits your requirements.

#### Workflow
1) User creates an issue using [Issue template](https://github.com/np-reddy/git-issue-ops/blob/main/.github/ISSUE_TEMPLATE/artifactory-repo-creation-request.md) and provides following details.
```
github.repo.name=devops1
arti.repo.type=local
arti.repo.package=ivy
```
2) [Actions workflow](https://github.com/np-reddy/git-issue-ops/actions/workflows/arti-object-creation.yml) is designed to trigger on issue opened (also, edited for testing the workflow) and checks for issue label **arti-repo** defined in our issue template.
3) Workflow can run on either GitHub-Hosted (ubuntu-latest) or self-hosted (ex.mylaptop).
4) Checkout the code from the repo using **actions/checkout@v1** 
5) Parse the issue description and extract gh repo, rt repo type, pkg type. Store them in separate files on runner workspace  so that these can be retrieved from other jobs whenever needed. Passing global env variables is also considered for passing variables but it is more complex (for me, at least 😉) and error prone.
6) Get the gh repo user permissions using collaborators api call and map them into admins, readers and writers based on the repo permissions, copy these to separate files on workspace.
7) Set the JFrog CLI env using **jfrog/setup-jfrog-cli@v1**
8) Prepare necessary object creation templates and set env varibales.
9) Create arti users and set default passowords and dummy emails (It is not possible to retrieve emails from GH if user chose the email to be private, not even from commit logs, I tried 🤷 )
Note: User validation if already present is not included, will be considered for enhancements.
10) Create arti groups and assign the users according to the user maps we have created earlier.
11) Create arti repo using env variables and arti repo template.
Note: Only local repos can be created for now.
12) Create arti permission target using permission template and env varibales, this will tag the repos and groups with required access. 
Note: Users are excluded from perm target as it is less complex to use groups.

Now, all the required objects are created in Artifactory and ready for use.

**Enhancements to be Planned**
1) Issue description validation for gh repo-name, arti repo-type, arti repo-pkg.
2) Validation for existing users and repos in artifactory
3) Creation of remote repos
4) Feedback to issue after object creation by using comment.
