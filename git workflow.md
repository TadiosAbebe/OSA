# Create Personal Access Token on GitHub
From your GitHub account, go to Settings → Developer Settings → Personal Access Token → Generate New Token (Give your password) → Fillup the form → click Generate token → Copy the generated Token, it will be something like ghp_sFhFsSHhTzMDreGRLjmks4Tzuzgthdvfsrta

Now follow the below method based on your machine:

For a Linux-based OS ⤴
For Linux, you need to configure the local GIT client with a username and email address,

$ git config user.name "your_github_username"
$ git config user.email "your_github_email"

Once GIT is configured, we can begin using it to access GitHub. Example:

$ git clone https://github.com/YOUR-USERNAME/YOUR-REPOSITORY
> Cloning into `YOUR-REPOSITORY`...
Username: <type your username>
Password: <type your password or personal access token (GitHub)
Now cache the given record in your computer to remembers the token:

$ git config --global credential.helper cache

If needed, anytime you can delete the cache record by:
$ git config --global --unset credential.helper
$ git config --system --unset credential.helper

# The Workflow

Test new features and configurations on the test virtual environment, once we see fit publish those configurations to the staging branch, pull configurations from the staging branch and deploy them on the staging environment, once feature and configurations are tested and benchmarked on the staging environment merge the staging branch to the master and repeat this cycle
