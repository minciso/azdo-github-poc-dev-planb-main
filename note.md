trigger:
  - none

pool:
  vmImage: windows-latest

resources:
  repositories:
    - repository: github_repo
      type: github
      name: minciso/azdo-github-poc
      endpoint: Azdo-GitHub-reposync          # ← the OAuth service connection name

stages:
  - stage: AzdoToGitHubSync
    jobs:
      - job: GitSync
        steps:

          # Step 1: Checkout ADO source repo
          - checkout: self

          # Step 2: Checkout GitHub repo with OAuth credentials persisted
          - checkout: github_repo
            persistCredentials: true

          # Step 3: Sync ADO content to GitHub and push
          - task: CmdLine@2
            displayName: 'Sync and Push to GitHub'
            inputs:
              script: |
                cd $(Pipeline.Workspace)\s\azdo-github-poc

                git config user.email "minciso15@gmail.com"
                git config user.name "minciso"

                git checkout -b $(Build.SourceBranchName)

                REM --- Copy files from ADO repo into GitHub repo ---
                REM Replace 'your-ado-repo' with the actual ADO repo folder name
                xcopy /E /Y /I "$(Pipeline.Workspace)\s\your-ado-repo\*" "." /EXCLUDE:exclude-list.txt

                git add --all
                git commit -m "Sync from Azure DevOps - $(Build.BuildNumber)"
                git push origin $(Build.SourceBranchName):azdo-github-poc-dev-planb