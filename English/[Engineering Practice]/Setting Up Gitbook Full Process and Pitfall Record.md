**Benefits of Converting a Notes Repository to GitBook:**

- Global text search.
- Tree-structured navigation for browsing notes.

------

# **Step 1: Prepare the Markdown Notes Repository**

All the following steps have been automated into a CLI tool, which you can directly run in the command line.

> Code location: https://github.com/Aki-98/Pre-Gitbook-Cli

------

**1. Convert all files to Markdown or reference them in Markdown**

GitBook can only render Markdown (.md) pages.

If you have PDF files, text files, or source code that you want to display on the GitBook website, here’s how:

- **For text files (txt)**: Convert them directly to Markdown format.
- **For PDF files and source code**: Use references to include them in Markdown files.

**2. Organize Images**

- You can reference online images in your Markdown files, and they will be displayed on the GitBook website. However, online images might become inaccessible over time. If this is a concern, download and save the images to the repository.
- If multiple computers access the same notes repository, image references with absolute paths might differ, causing GitBook to fail in rendering the images. In this case, rewrite the absolute paths or use relative paths instead.
- If your notes contain photos of physical books, these images may be large. Moving such images within the repository can unnecessarily increase the size of the `.git` file. To address this, compress the images to reduce size without significant data loss.

**3. Prepare `SUMMARY.md`**

The `SUMMARY.md` file is used for the tree-structured navigation on the right-hand side of GitBook.

While the `gitbook init` command can generate a `SUMMARY.md`, it often comes with various bugs and cannot generate a complete summary based on all Markdown files.

**4. Prepare `book.json`**

The `book.json` file is used to configure the GitBook website, including the title, copyright information, and plugins (e.g., global search).

Here’s my configuration for reference:
https://github.com/Aki-98/CSBlogGitbook/blob/main/book.json

**Configuration Explanation:**

```json
{
    "root": "/", 
    "gitbook": "3.2.3", 
    "title": "CS Blog", // Website title
    "page": {
        "title": "CS Blog" // Website title
    },
    "author": "aki", // Author
    "variables": {
        "authorName": "aki" // Author name
    },
    "plugins": [
        "accordion", // A plugin for collapsible content or expandable sections (accordion effect)
        "search-pro", // An improved global search plugin providing more powerful search functionality
        "-search", // Disables the default search plugin (GitBook's built-in search)
        "tbfed-pagefooter" // A plugin for adding copyright statements and modification time to the page footer
    ],
    "pluginsConfig": {
        "tbfed-pagefooter": { // Adds content to the footer of each file
            "copyright": "Copyright &copy Aki 2024", // Copyright statement
            "modify_label": "modified time:", 
            "modift_format": "YYYY-MM-DD HH:mm:ss" // Format for the modification time
        }
    }
}
```

# Step 2: Prepare the Gitbook Setup Environment

*Alternatively, you can deploy GitBook via CI/CD to skip this step.*



I followed some online tutorials and encountered many issues.

> This tutorial is too simple. You can take a look to get a general idea, but don't follow its procedures:
>
> https://tonydeng.github.io/gitbook-zh/
>
> This one is more detailed and better:
>
> https://1927344728.github.io/fed-knowledge/tools/Gitbook/

**0. Preparing for pitfalls**

At first, I directly downloaded and installed from the official website:

> Official website: https://nodejs.org/en

However, the version from the official website (v18.x) is too new, causing various adaptation issues with gitbook. Later, I downgraded to v.10.x to use gitbook normally.

I encountered an error when installing gitbook-cli after installing v18.x version of nodejs and calling gitbook init in the command line:

```
/home/travis/.nvm/versions/node/v12.18.3/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287
      if (cb) cb.apply(this, arguments)
                 ^
TypeError: cb.apply is not a function
    at /home/travis/.nvm/versions/node/v12.18.3/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287:18
    at FSReqCallback.oncomplete (fs.js:169:5)
```

> Solution: https://stackoverflow.com/questions/64211386/gitbook-cli-install-error-typeerror-cb-apply-is-not-a-function-inside-graceful

You need to update the graceful-fs version.

When I tried to update with npm, I encountered another issue where any command would result in an error "ERR! Cannot read property 'insert' of undefined".

> Solution: https://github.com/npm/cli/issues/4876

You need to change the mirror to [https://registry.npmmirror.com](https://registry.npmmirror.com/). If you also need to set a proxy, there is a tutorial here:

> Setting npm proxy: https://blog.csdn.net/yanzi1225627/article/details/80247758

After updating, calling any gitbook command resulted in no response, no error, and no information. The local directory remained unchanged. I then downgraded nodejs to 10.x.

During this process, there was also an issue that gitbook and gitbook-cli could not coexist. You need to uninstall gitbook to use gitbook-cli (the command-line tool for gitbook). After installing gitbook-cli, the first time you call the gitbook command, like checking the version with `gitbook -V`, gitbook-cli will install the corresponding version of gitbook.

Since I got stuck at the "install gitbook" step every time I called the gitbook command and checked online for a solution, I found that you can clone gitbook's github repository and then install and link it yourself:

```
git clone https://github.com/GitbookIO/gitbook.git
cd gitbook
npm install
npm link
```

Firstly, the instructions in the gitbook repository say to install with `bun` instead of `npm`. Secondly, this installation is equivalent to `npm install gitbook`, which conflicts with gitbook-cli. Therefore, this method is incorrect.

Here is the correct method:

**1. Install nvm (Node.js version manager) and Node.js v10.x**

Choose one based on your system:

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

bash
Copy code
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

Install Node.js v10.x and use it:

```
nvm install 10
nvm use 10
```

I also encountered an issue here. In the above steps, the npm version compatible with Node.js v10.x will be installed. However, I previously used v18.x. After switching to Node.js v10.x, the npm version did not change. Any npm command would result in an error. Later, I switched back to v18, uninstalled npm, which uninstalls the latest version of npm, and then switched to v10.x, allowing me to use the older version of npm.

```
nvm use 18
npm uninstall npm -g
```

The `-g` means the operation is carried out globally, which will be used in the following commands.

**2. Install and use gitbook-cli & gitbook**

```
npm install gitbook-cli -g
```

The `npm install -g` command is used to globally install Node.js modules. This means the installed modules will be placed in a specific directory, usually the global `node_modules` directory of your system, not in the `node_modules` directory of the current project.

If you don't use the `-g` (global) parameter in the `npm install` command, the command will install the specified package in the `node_modules` folder of the current project, not in the global environment.

```
gitbook -V
```

Check the gitbook version. The first time you call any gitbook command, it will attempt to install gitbook.

Running `gitbook -V` again will show the versions of gitbook-cli and gitbook.

Note: Do not install gitbook with `npm install gitbook` directly; use gitbook-cli instead.

**3. Preview the Gitbook website and make adjustments**

Switch the command line to the root directory of your MD source files and run the following command to automatically generate README.md and SUMMARY.md:

```
gitbook init
```

Note: Do not include the character "#" in the directory names and filenames under the MD files, meaning the generated SUMMARY.md should not contain the "#" character. If it does, it will cause the links in the gitbook page's left navigation bar to not work.

Also, do not have consecutive curly braces in the MD files, like {{ and }}.

> Related instructions: [https://wkcom.gitee.io/gitbook%E7%9A%84%E4%BD%BF%E7%94%A8/3%E3%80%81gitbook%E8%BF%9E%E7%BB%AD%E5%A4%A7%E6%8B%AC%E5%8F%B7%E7%9A%84%E8%A7%A3%E5%86%B3%E6%96%B9%E5%BC%8F.html](https://wkcom.gitee.io/gitbook的使用/3、gitbook连续大括号的解决方式.html)

Adding a space between the curly braces will resolve the issue.

Preview the gitbook website:

```
gitbook server
```

Open the local port in the browser to view the page:

```
http://localhost:4000/
```



# Step 3: Set up .io online website

*Alternatively, you can deploy GitBook via CI/CD to skip this step.*



There are many solutions; I chose to directly upload the static website to Github.

Generate the gitbook static website, run the following command in the root directory of your MD source files:

```
gitbook build
```

This will generate a `_book` directory, containing the static website code.

As long as the repository has a `gh-pages` branch, you can access Gitbook Pages with the following link:

```
https://githubusername.github.io/reponame
```

So the steps are:

1. Create an empty Github repository.
2. Check out a `gh-pages` branch, upload the static website.
3. I need multiple gitbooks, so I created a subfolder in the root directory and uploaded the corresponding static website in this subfolder.

Then, I can access multiple Gitbooks with the following paths:

```
https://githubusername.github.io/reponame/subfoldername1
https://githubusername.github.io/reponame/subfoldername2
```



# **Step 4: GitBook CI/CD**

You can directly copy the following file into your notes repository:

https://github.com/Aki-98/CSBlogGitbook/blob/main/.github/workflows/deploy.yml

If the branch of your notes repository is not `main`, you need to rewrite the triggering condition in the first part of the `.yml` file:

```yaml
on:
  push:
    branches: 
      - {your_branch}
```

Make sure the file is placed in the correct path:
`/.github/workflows/deploy.yml`

Go to **Settings > Developer Settings > Personal access tokens > Fine-grained tokens > Generate New Token** in your GitHub personal account.

For permissions, I set them to the maximum since I am unsure about the minimum required permissions. Feel free to investigate and configure as needed.

Next, navigate to your notes repository's **Settings > Secrets and variables > Actions**, and create a secret named `PERSONAL_TOKEN`. Copy the token you generated earlier into this field.

Now, every time you push to the repository, a GitBook webpage will be automatically generated and published to the `gh-pages` branch of your repository.

You can access the webpage using the following URL format:

```
https://{your_github_name}.github.io/{repo_name}
```

**Detailed Explanation of `deploy.yml`**

Who would’ve thought writing such a simple YAML file could involve so many pitfalls...

```yaml
# Name of the CI/CD task
name: Deploy GitBook

# Trigger condition: When a user pushes to the `main` branch of this repository
on:
  push:
    branches:
      - main

# CI/CD jobs
jobs:
  deploy:
    # Platform for running the CI/CD task: Latest version of Ubuntu
    runs-on: ubuntu-latest
    steps:
      # Step 1: Check out the branch of the notes repository
      - name: Checkout code
        uses: actions/checkout@v3

      # Step 2: Set up a Node.js 12 environment. A lower version is chosen for GitBook compatibility.
      - name: Setup Node.js 12
        uses: actions/setup-node@v3
        with:
          node-version: '12'

      # Step 3: Install `graceful-fs` in advance to avoid errors (refer to Step 2 for details)
      - name: Install Latest graceful-fs
        run: npm install graceful-fs@4.2.0 -g

      # Step 4: Install the GitBook CLI tool
      - name: Install GitBook CLI
        run: npm install gitbook-cli@2.1.2 -g

      # Step 5: Install GitBook using the CLI tool
      - name: Install GitBook
        run: gitbook install

      # Step 6: Render and generate the GitBook webpage based on the notes
      - name: Build GitBook
        run: gitbook build

      # Step 7: Deploy the GitBook webpage to the `gh-pages` branch of the repository
      # Uses the `peaceiris/actions-gh-pages` GitHub Action. For more usage details, refer to:
      # https://github.com/peaceiris/actions-gh-pages
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.PERSONAL_TOKEN }} # Use the `PERSONAL_TOKEN` from this repository
          publish_dir: ./_book # Use the files from the `_book` directory
          publish_branch: gh-pages # Publish to the `gh-pages` branch
```
