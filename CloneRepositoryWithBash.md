# How to clone a GitHub Repository with `Bash`

## Structure

1. [Generating and add a new SSH key to the ssh-agent and GitHub](#generating-and-add-a-new-ssh-key-to-the-ssh-agent-and-github)
2. [Generate GitHub token](#generate-a-github-token)
3. [Latest Version](#clone-the-latest-version-from-the-repository)
4. [Latest Release](#clone-the-latest-release-from-the-repository)
5. [Tagged Version](#clone-a-tagged-version-from-the-repository)

## Generating and add a new SSH key to the ssh-agent and GitHub

1. Paste the text below, substituting in your GitHub email address.

    ```shell
    $ ssh-keygen -t ed25519 -C "your_email@example.com"
    ```

2. When you're prompted to "Enter a file in which to save the key", you can press Enter to accept the default file location.

    ```shell
    > Enter a file in which to save the key (/home/YOU/.ssh/ALGORITHM):[Press enter]
    ```

3. Start the ssh-agent in the background.

    ```shell
    $ eval "$(ssh-agent -s)"
    > Agent pid 59566
    ```

4. Add your SSH private key to the ssh-agent.

    ```shell
    $ ssh-add ~/.ssh/id_ed25519
    ```

5. Add your SSH public key to GitHub.

    1. Show your SSH public key.

        ```shell
        $ cat ~/.ssh/id_ed25519.pub
        ```

    2. Open Github in the web browser.
    3. In the upper-right corner of any page, click your profile photo, then click Settings.
    4. In the "Access" section of the sidebar, click **SSH and GPG keys**.
    5. Click **New SSH key** or **Add SSH key**.
    6. In the "Title" field, add a descriptive label for the new key.
    7. Paste your key into the "Key" field.
    8. Click Add SSH key.

## Generate a GitHub token

1. Go to the GitHub [settings](https://github.com/settings/profile)
2. Go to the point "Developer settings"
3. Go to the point "Personal Access tokens"
4. Then on Fine-grained tokens
5. Now choose "Generate new token"
6. Give that token a name and a expiration time
7. Choose the needed "Repository access"

    **If you selected the point "Only select repositories"**
    1. Add to the "Repository permissions" the point "Contents"

        (This allow you to access the contents, commits, branches, downloads, releases, and merges from the selected repositories)
8. Click "Generate token" to generate yout token
9. Save the shown token safe.

    **NOTE: This token will only shown one time to you.**

## Clone the latest version from the repository

To use this script you have to install git with this command.

```shell
sudo apt-get install git
```

1. Create a bash file with the following code.

    ```bash
    #!/bin/bash

    git clone git@github.com:[USER]/[REPO].git --branch [BRANCH]
    ```

2. You need only to replace *``[USER]``* with your username, *``[REPO]``* with the repositiory name and *``[BRANCH]``* with the branch name.

## Clone the latest release from the repository

To use this script you have to install git and jq with this command.

```shell
sudo apt-get install git jq
```

1. Create a bash file with the following code.

    ```bash
    #!/bin/bash

    json=`curl \
    -H "Accept: application/vnd.github+json" \
    -H "Authorization: Bearer [YOUR_TOKEN]"
    -H "X-GitHub-Api-Version: 2022-11-28" \
    https://api.github.com/repos/[USER]/[REPO]/releases/latest`

    tag=`echo $json | jq -r '.tag_name'`
    git clone git@github.com:[USER]/[REPO].git --branch [BRANCH]

    git checkout $tag
    ```

2. You need only to replace *``[USER]``* with your username, *``[REPO]``* with the repositiory name, *``[BRANCH]``* with the branch name and *``[YOUT_TOKEN]``* with your token for that repository.

    **NOTE: The token need the permission "Contents"**

    If you need help to generate your token [look here](#generate-a-github-token)

## Clone a tagged version from the repository

1. Create a bash file with the following code.

    ```bash
    #!/bin/bash

    git clone git@github.com:[USER]/[REPO].git --branch [BRANCH]

    git checkout [TAG]
    ```

2. You need only to replace *``[USER]``* with your username, *``[REPO]``* with the repositiory name, *``[BRANCH]``* with the branch name and *``[TAG]``* with the tag which you want to clone.
