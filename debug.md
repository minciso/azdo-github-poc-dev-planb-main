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

          # Step 3 (optional, remove after first run): Debug - verify checkout folder names
          #- task: CmdLine@2
          #  displayName: 'Debug - List checkout folders'
          #  inputs:
          #    script: |
          #      dir "$(Pipeline.Workspace)\s"

          # Step 4: Sync ADO content to GitHub and push
          - task: CmdLine@2
            displayName: 'Sync and Push to GitHub'
            inputs:
              script: |
                cd $(Pipeline.Workspace)\s\azdo-github-poc

                git config user.email "minciso15@gmail.com"
                git config user.name "minciso"

                git checkout -b $(Build.SourceBranchName)

                REM --- Copy files from ADO repo into GitHub repo ---
                REM Replace 'MyADORepoName' with the actual ADO repo folder name
                REM (run the debug step above to find the correct folder name)