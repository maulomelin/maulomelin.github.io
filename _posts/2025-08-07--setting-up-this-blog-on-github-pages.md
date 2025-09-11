---
title: Setting Up This Blog on GitHub Pages
---
# {{ page.title }}

<span style="color: #999">_Published: {{ page.date | date: "%Y-%m-%d" }}_</span>

This is a Jekyll site hosted on GitHub Pages. In this post, I explain why I chose this as my personal documentation system, provide a step-by-step guide to creating a similar site, and share some reflections on using AI coding tools for this project.

## TL;DR

* **Why GitHub Pages?** First, it is a free hosting feature on a well-supported platform - suitable for my blog. In addition, site content sits in a Git repository and is published automatically when pushed to GitHub, which fits my workflows and gives me reliable, versioned backups. Finally, with the Jekyll static site generator I get full control over all site files, a clean separation of content and design, and no vendor lock-in - making it easy to update the UI or switch providers at any time.

* **How GitHub Pages?** Follow this [simple shell script Gist](https://gist.github.com/maulomelin/dc590f733ac66688ee91f14acc6b290d) to build a Jekyll site hosted on GitHub Pages.

* **What About AI Coding Tools?** While they provide great speed improvements, effective guidance is critical. They are a complement, not a replacement, to good architecture, design, and planning.

## Why GitHub Pages?

GitHub Pages is a GitHub feature that hosts a static website from a repository. You create a GitHub Pages site by configuring a GitHub repository as its publishing source and specifying a theme that controls the site's UX layout and styling. When updates are pushed to that repository, a GitHub Actions workflow is triggered that runs a build process on the source code in the repo, and then deploys the resulting static files to GitHub Pages' hosting environment.

Let's unpack this to explain my decision after having explored several options:

* **GitHub Pages is a GitHub feature.** GitHub has a huge user base and hosts a ton of open-source projects. This means there are a lot of [popular GitHub Pages](https://github.com/collections/github-pages-examples) that serve as examples, and there is a large community for support.

* **GitHub Pages hosts static websites.** GitHub as a hosting service is very attractive because it is free for static websites and Microsoft has an interest in ensuring GitHub remains a solid platform. A static website is really all I need for my personal documents.

* **Repositories are publishing sources.** I spend a good amount of time creating material that is version-controlled with Git and shared on GitHub. Creating and maintaining my own documentation using the same tools and workflow I already use is a huge boon. No context switching. No need to navigate to a separate tool or UI to work on my personal docs. Efficiency. I can perform all the publishing tasks I want within my current workflows:

    * Want to write a new post? Create a new doc in your IDE.
    * Want to work on multiple draft posts? Do it on a new branch.
    * Want to schedule a post's publish date? Set the date as a page variable and tweak the templates.
    * Want to preview the latest versions of any draft document? Run a local build.
    * Want to update the UX of the site? Update any theme files and push the changes to GitHub.
    * Broke the site with an update to a template file? Roll back changes and revert to a working version.

* **Repositories are stored locally and on GitHub.** Have you ever lost important data stored in an online service because of a snafu? Yeah...not an experience one boasts about. Storing my content on GitHub and on local repositories across multiple machines, as I usually have, is a good safety net supporting the [3-2-1 backup rule](https://en.wikipedia.org/wiki/Backup#3-2-1_Backup_Rule), regardless of a service provider's disaster recovery plan.

* **A build process runs automatically on updates to the repo.** A build process for a static website is of great value to me. Having pre-built static pages eliminates the need for a back-end database to hold any data or server-side processing to generate pages on the fly, which makes the site simpler to set up, easier to maintain, and faster to serve up.

    GitHub Pages can be configured to automatically run a build and publish a site when changes are pushed to a repo. This automated workflow makes the publishing aspect efficient, consistent, and reliable. I can also trigger local or remote builds at any time, enabling me to preview updates or troubleshoot any build issues on demand.

    This biggest benefit of this model, however, is that it gives me full access to every file created or generated, which means I am not locked into any provider, I am not limited by theme restrictions or drag-and-drop GUIs, and I can move all my content at will.

* **The build process is configurable.** GitHub Pages' build process is very flexible: it supports [dozens of static site generators (SSGs)](https://github.com/collections/static-site-generators) like Jekyll, VuePress, Next.js, and others. This allows me to choose a framework based on my preferred language, available plugins, and personal preference. And while the tinkerer in me gets excited about the possibility of trying out different frameworks in the future as requirements change, the pragmatist in me recognizes that the default build process is good enough for my current needs.

* **The default build process uses Jekyll.** Jekyll is an SSG that takes content files written in Markdown and combines it with page template files written in HTML/CSS/JavaScript/Liquid to produce a complete static website. These are all languages I am well-versed with. The [GitHub Pages documentation](https://docs.github.com/en/pages) and the [Jekyll documentation](https://jekyllrb.com/) are well organized and easy to follow - I was able to quickly prototype and evaluate a Jekyll site. This was a big plus for me - the lack of good documentation and/or test environments often hinder a product from selling itself.

* **Jekyll site content is written Markdown and other familiar languages.** On a Jekyll site, content files are written in Markdown - a simple markup language used to format plain text - or HTML.

    The most recent iteration of my personal docs were written in HTML - I had transitioned away from using Word because I was tinkering with UI frameworks and found it easier to switch within my IDE and use HTML to write my documents. I found the experience of writing content without any predefined formatting or tied to a specific program liberating.
    
    Moving to Markdown for my next iteration feels natural and straight-forward. It is more lightweight and readable than HTML, and it plays nice with HTML - I can inject HTML/CSS/JavaScript code into a Markdown document without breaking its meaning or the Jekyll build tools, and it just works. Mostly. Having the flexibility to do this in a streamlined fashion is invaluable to me - I can use or experiment with different client-side frameworks and technologies (e.g., jQuery, TailwindCSS, HTMX, Vue, Angular, React) on different pages without having to refactor the entire site.

* **Content and Presentation are deliberately separate in a Jekyll project.** One of the design principles of Jekyll as a tool is to keep content and presentation separate: Content is stored in some folders (`_posts/`, `images/`, etc.), and presentation is handled through template files in other folders (`_layouts/`, `_includes/`, `assets/`, etc.). At build-time, Jekyll combines these to generate the complete site in a single folder (`_site/`) which is then published.

    There are two big advantages to this design:
    1. The basic GitHub Pages setup allows me to get started right away. I can simply start checking in new post files and the blog will be automatically updated with whatever presentation defaults there are. However, I also wanted to customize the default site...
    1. This separation makes it very easy to update the site's look and feel by simply switching themes or modifying template files, without touching the content itself - I can essentially "re-skin" the site in minutes if I find a theme I like.

* **GitHub Pages allows multiple sites per account.** As an added bonus, GitHub Pages allows a single *User Site* to be created for an account (at `https://{username}.github.io`) and separate *Project Sites* for each repository in that account (at `https://{username}.github.io/{repo_name}`), which means I can apply everything I learn from building out my personal docs to document any project repo, like these [popular GitHub Pages](https://github.com/collections/github-pages-examples).

## How GitHub Pages?

This section walks you through creating a simple Jekyll site on GitHub Pages, with the ability to run local builds and customize the site's template files. It is part runbook, the result of many prototypes and experiments; and part guide, with notes that explain some design decisions. As such, there is plenty of room to do or explain things differently - and better.

1. Install Jekyll on your local environment to run local Jekyll builds.

    ```shell
    #!/bin/zsh

    # Verify environment.
    if [ -z "${ZSH_NAME}" ]; then   # Fail fast if not running in zsh.
        echo "This is a pseudo-zsh script. Adjust for other shells."
        exit 1
    fi

    # Install Homebrew to install Jekyll and Bundler.
    if ( ! brew -v ); then          # Install Homebrew.
        /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    else
        brew update                 # Update Homebrew.
        brew upgrade                # Install the newest versions of all packages.
    fi

    # Install Jekyll and Bundler to run local Jekyll builds.
    if ( jekyll -v ); then          # Update Jekyll and Bundler.
        gem update jekyll           # Update Jekyll gem to the latest version.
        gem update bundler          # Update Bundler gem to the latest version.
    else                            # Install Jekyll and Bundler.
        brew install chruby         # Install Ruby version manager.
        brew install ruby-install   # Install Ruby installer.
        ruby-install ruby 3.4.1     # Install Ruby for Jekyll.
        # Update the zsh configuration file to use chruby by default.
        echo "source $(brew --prefix)/opt/chruby/share/chruby/chruby.sh" >> ~/.zshrc
        echo "source $(brew --prefix)/opt/chruby/share/chruby/auto.sh" >> ~/.zshrc
        echo "chruby ruby-3.4.1" >> ~/.zshrc
        source ~/.zshrc             # Apply updates (alternate: `exec zsh`).
        gem install jekyll          # Install the latest Jekyll gem.
        gem install bundler         # Install the latest Bundler gem.
    fi

    # Check versions for compatibility.
    ruby -v                         # Check version: gem "ruby", "3.4.1"
    jekyll -v                       # Check version: gem "jekyll", ">=4.4.1"
    bundler -v                      # Check version: gem "bundler", ">=2.6.9"

    # Run Homebrew diagnostics to check for issues.
    brew doctor                     # Run Homebrew diagnostics, as an FYI.
    ```

    <details markdown="1">
    <summary>More Info</summary>

    * This pseudo-script, like others I post, is designed to be followed by hand. It assumes familiarity with shell scripting, and with the Z Shell syntax, in particular. For example, `if ( jekyll -v ) ; then...` is equivalent to running the command `$ jekyll -v` with its exit status indicating whether Jekyll is installed or not. Guides I write automated scripts for are not perfect but more robust (i.e., configurable, modular, idempotent, structured logging, etc.) and kept in [the `bin/` folder of my shell-scripts repository](https://github.com/maulomelin/shell-scripts/tree/main/bin).

    * About what this script installs:

        * Homebrew is a package manager used to download software packages.
        * Ruby is a programming language installed using Homebrew.
        * A Ruby gem is a packaging format for distributing Ruby programs.
        * Bundler is a Ruby gem that manages gem dependencies and keeps a consistent environment.
        * Jekyll is a Ruby gem that builds static websites.

    * See the [Jekyll Installation docs](https://jekyllrb.com/docs/installation) and the [Jekyll Step by Step Tutorial](https://jekyllrb.com/docs/step-by-step) for more details.

    </details>

1. Create a GitHub Pages repo and clone it locally to build the site on your machine.

    ```shell
    #!/bin/zsh

    # Script settings.
    readonly _USERNAME=$(gh api user -q ".login")
    readonly _REPO_NAME="${_USERNAME}.github.io"
    readonly _REPO_DESCRIPTION="Personal blog source."
    readonly _DEV_ROOT="${HOME}/_dev"
    readonly _WORKSPACE_DIR="${_DEV_ROOT}/repos/github.com/${_USERNAME}"
    readonly _TMP_DIR="_tmp"
    readonly _JEKYLL_GITIGNORE="https://raw.githubusercontent.com/github/gitignore/HEAD/Jekyll.gitignore"
    readonly _MACOS_GITIGNORE="https://raw.githubusercontent.com/github/gitignore/HEAD/Global/macOS.gitignore"
    if [[ "${_REPO_NAME}" == "${_USERNAME}.github.io" ]]; then
        readonly _REPO_HOMEPAGE="https://${_REPO_NAME}/"                         # User site.
    else
        readonly _REPO_HOMEPAGE="https://${_USERNAME}.github.io/${_REPO_NAME}/"  # Project site.
    fi

    # Move to the workspace directory to create a repo.
    mkdir -p "${_WORKSPACE_DIR}"    # Create intermediate directories, if needed.
    cd "${_WORKSPACE_DIR}"          # Move to the directory.

    # Create a repo for the GitHub Pages site and clone it locally.
    gh repo create "${_REPO_NAME}" --public --gitignore Jekyll --clone --description "${_REPO_DESCRIPTION}" --homepage "${_REPO_HOMEPAGE}"

    # Move to the project directory.
    cd "${_REPO_NAME}"

    # Update the `.gitignore` file with custom + Jekyll + macOS entries.
    (   echo "### Source: Custom\n";                 echo "${_TMP_DIR}/"; \
        echo "\n### Source: ${_JEKYLL_GITIGNORE}\n"; curl -fsSL ${_JEKYLL_GITIGNORE}; \
        echo "\n### Source: ${_MACOS_GITIGNORE}\n";  curl -fsSL ${_MACOS_GITIGNORE} \
    ) > GITIGNORE
    mv GITIGNORE .gitignore

    # Confirm the remote GitHub fetch/push URLs.
    git remote -v
    ```

    <details markdown="1">
    <summary>More Info</summary>

    * We create the repos first because the name is needed to configure Jekyll for GitHub Pages later on.
    
    * `~/_dev` is the root directory for all our development projects.

    * The `$ gh api user -q ".login"` command retrieves the current GitHub username. Using this command helps us set up the site on different GitHub accounts. It also genericizes the documentation.

    * The `$ gh repo create ...` command creates a remote repository and clones it to a local directory of the same name.

        * The repo `{username}.github.io` is required for the user site `https://{username}.github.io`.

        * To keep things simple, GitHub Pages will be configured to publish from the `main` branch and from the `/ (root)` folder, so there is no need to create, checkout, or clear a new branch.

        * The `--public` flag creates a public repository. We need this because GitHub Pages is only available for public repositories on free GitHub accounts and ours is a free account.

        * The `--gitignore Jekyll` option adds a `.gitignore` template file to a Jekyll project. We rewrite it next, but we include it to describe how to access other gitignore templates:

            * For gitignore keywords run `$ gh repo gitignore list`.

            * To view other templates files visit `https://github.com/github/gitignore`.

            * Downloading the Jekyll gitignore file programmatically is equivalent to the command `$ curl https://raw.githubusercontent.com/github/gitignore/HEAD/Jekyll.gitignore -o .gitignore`.

        * The `--clone` flag clones the remote repo to a local directory with the same name.

        * The `--description` option adds a description in the "About" section of the repo's GitHub page.

        * The `--homepage` option adds a URL in the "About" section of the repo's GitHub page.

    * The final set of `echo`, `curl`, and `mv` commands create a single `.gitignore` file that includes custom, Jekyll, and macOS entries.

        * The macOS entries are for the macOS environment we are on. Change it for a different OS.

        * The custom entry is a temporary directory, `_tmp/`, that will be used to store a reference library of Jekyll themes.

            * Having it in the `.gitignore` file ensures it won't be saved with the repo.

            * Having it prefixed with `_` ensures Jekyll does not include it in the build because [Jekyll ignores folders that start with `_`](https://jekyllrb.com/docs/structure/).

    * I decided to protect all content I create on this site under an exclusive copyright, while allowing free use of the site's source code. The repository's README.md file that describes this will be created in the next steps. To publish the site under a global license, add a LICENSE file when creating the repo:

        ```shell
        #!/bin/zsh

        # Script settings.
        readonly _CONTACT_INFO="Mauricio Lomelin &lt;maulomelin@gmail.com&gt;"
        readonly _LICENSE="MIT"

        # Create a GitHub repo and clone it locally.
        gh repo create "${_REPO_NAME}" --public --license {$_LICENSE} --gitignore Jekyll --clone --description "${_REPO_DESCRIPTION}" --homepage "${_REPO_HOMEPAGE}"

        # Update contact info in the LICENSE file.
        sed -i "" "s/${_USERNAME}/${_CONTACT_INFO}/" LICENSE
        ```

        * The `--license MIT` option adds a LICENSE file to the repo with the MIT License. The installed license file pre-populates the copyright line with the current year and with the current GitHub username.

            * For license keywords run `$ gh repo license list`.
            
            * To view other license templates visit [https://choosealicense.com](https://choosealicense.com) or [https://github.com/licenses/license-templates/](https://github.com/licenses/license-templates/). These templates will generally have a placeholder for various fields (e.g., `[year]`, `{%raw%}{{year}}{%endraw%}`, `[fullname]`, `{%raw%}{{organization}}{%endraw%}`, etc.), which must be replaced with your own information.

        * The `$ sed ...` command changes the GitHub username in the LICENSE file with more specific contact info. For other licenses, check what fields you may want to replace.

    * As a side note, while many of these steps could be done manually in the GitHub.com UI, using the [GitHub CLI](https://cli.github.com/) and shell commands is faster and easier. For comparison, see how the script commands above compare with their manual, GUI-based counterparts:

        1. Open a browser, sign in to [https://github.com](https://github.com), and create an empty, public repository with a Jekyll gitignore and an MIT License.
        1. Clone the remote repository to a local directory.
        1. Update the `.gitignore` file to include a custom entry and entries from the Jekyll and macOS `.gitignore` templates.
        1. Open the LICENSE file in the local repo and update the copyright contact info.

        This example shows why I often prefer to write shell scripts over documenting GUI-based instructions. Scripts are less ambiguous and easier to reuse than following manual steps. Fortunately, many IDEs now integrate with CLIs through built-in terminals and extensions, making scripting easier to manage and execute. However, scripting is a double-edged sword: they're a powerful tool if you're proficient, but if you're not you could cause serious problems - always double-check and test your scripts!

    </details>

1. Create a `README.md` file in the project root to provide copyright and licensing terms.

    ```shell
    #!/bin/zsh

    # Script settings.
    readonly _CONTACT_INFO="Mauricio Lomelin &lt;maulomelin@gmail.com&gt;"

    # Create a README.md file with copyright and licensing terms.
    cat <<CONTENT > README.md
    # About

    This is my personal blog - a collection of notes, tutorials, and code snippets.

    It's built with Jekyll and hosted on GitHub Pages.

    ## Copyright and Licensing

    The content in the \`_posts/\` and \`images/\` directories of this project is the copyright of ${_CONTACT_INFO}. Do not use without permission.

    All other files and content are licensed under the MIT License.
    CONTENT
    ```

    <details markdown="1">
    <summary>More Info</summary>

    * For projects with varied copyright and licensing requirements, there are different ways to specify which terms apply where. We chose a directory-based approach because it aligns with how our content is organized. See the [ChooseALicense.com repo](https://github.com/github/choosealicense.com) and the [SPDX Specification](https://spdx.dev/learn/handling-license-info/) for more info.

    </details>

1. Create a `Gemfile` file in the project root so that local builds match GitHub Pages builds.

    ```shell
    #!/bin/zsh

    # Create a `Gemfile` file to match local builds with GitHub Pages builds.
    cat <<CONTENT > Gemfile
    source "https://rubygems.org"
    gem "github-pages", group: :jekyll_plugins
    CONTENT
    ```

    <details markdown="1">
    <summary>More Info</summary>

    * The resulting file should look like this:

        ```ruby
        # filename: Gemfile

        source "https://rubygems.org"
        gem "github-pages", group: :jekyll_plugins
        ```

    * The `Gemfile` file contains a description of all gem dependencies for a Ruby program.

        * Gems are installed from the gem hosting service specified in the `sources` setting. Bundler installs gems from `https://rubygems.org` by default, but I prefer being explicit in my configuration files.

        * The CLI command `$ bundle init` could have been used to create a default `Gemfile` file, but it would still need to be edited to add the GitHub Pages gem and remove any incompatible gems. Copy/pasting the two lines above is easier.

        * The CLI command `$ bundle install` is used to install all project gems listed in its `Gemfile` file. We'll cover this in more detail when it's time to run this command.

        * For more information, run `$ jekyll new .` inside a temp directory to create a default Jekyll site and view its default `Gemfile` file.

    * The `github-pages` gem sets up and maintains the local Jekyll environment in sync with GitHub Pages' Jekyll environment. It includes the `jekyll` gem and all other [GitHub Pages Dependency versions](https://pages.github.com/versions/), which allows us to build and test the site locally.

        * `github-pages` is a wrapper gem that includes all gems needed to make your local Jekyll build environment match the GitHub Pages build environment. See [https://pages.github.com/versions.json](https://pages.github.com/versions.json) for all gem dependencies.

        * Run `$ bundle update github-pages` periodically to update the local `github-pages` gem to stay in sync with the `github-pages` gem on the GitHub Pages server.

        * For more information, see the [GitHub Pages Ruby Gem repo](https://github.com/github/pages-gem).

    </details>

1. Create a minimal `_config.yml` file in the project root to configure Jekyll for GitHub Pages.

    ```shell
    #!/bin/zsh

    # Create a minimal `_config.yml` file to configure Jekyll for GitHub Pages.
    cat <<CONTENT > _config.yml
    title: Personal Documents
    description: A collection of notes, tutorials, and code snippets.
    repository: ${_USERNAME}/${_REPO_NAME}
    CONTENT
    ```

    <details markdown="1">
    <summary>More Info</summary>

    * The resulting file should look like this:

        ```yaml
        # filename: _config.yml

        title: Personal Documents
        description: A collection of notes, tutorials, and code snippets.
        repository: maulomelin/maulomelin.github.io
        ```

    * The `_config.yml` file is Jekyll's main configuration file. It defines site-wide settings, global variables, and controls how the static website is built.

        * This is the minimal `_config.yml` file for a GitHub Pages site. The [basic Jekyll setup instructions](https://jekyllrb.com/docs/step-by-step) do not require this file (a basic Jekyll setup uses system defaults). The `github-pages` gem, however, does require a `_config.yml` file with at minimum the settings shown above to build the site.

        * For more information, run `$ jekyll new .` inside a temp directory to create a default Jekyll site and view its default `_config.yml` file.

    </details>

1. Add the Jekyll theme "Primer" to `_config.yml` to set up a minimal UX for the site.

    ```shell
    #!/bin/zsh

    # Add the Jekyll theme "Primer" to `_config.yml` to give a minimal UX to the site.
    cat <<CONTENT >> _config.yml
    theme: jekyll-theme-primer
    CONTENT
    ```

    <details markdown="1">
    <summary>More Info</summary>

    * The resulting file should look like this:

        ```yaml
        # filename: _config.yml

        # ...
        theme: jekyll-theme-primer
        ```

    * A *Jekyll theme* is a collection of template files that define the look & feel (UX) of the website. Jekyll uses these files at build-time to generate a complete, styled, static website.

        * A project's theme can be implemented in different ways:

            * **Gem-Based Themes.** A gem-based theme is a Jekyll theme packaged as a Ruby gem. To use one, you simply install the gem and specify that theme in the project. When Jekyll builds the site it will pull the theme's template files from the gem. These are good to use when you want to use a theme as-is.

            * **Local Themes.** A local theme is a Jekyll theme where all of the template files are in a project's directory. To use this, you simply create template files for your site and place them inside your project directory. When Jekyll builds the site it will use those template files to generate all site pages. These are good to use when you want to build your own custom theme and want complete control.

            * **Theme Overrides.** A theme override lets you make targeted changes to a gem-based theme. To override a file, you first specify the theme in the project. Then, you place a new version of the template file to override in the project directory using the same name and relative path as the file in the theme. When Jekyll builds the site, it will use the local version of any template files it finds instead of the ones from the theme. This is a good approach to follow if you want to customize parts of an existing theme.

        * For a GitHub Pages site, a gem-based theme can be specified in different ways:

            * **Use a theme that is pre-bundled with `github-pages`.** The `github-pages` gem includes a bunch of dependencies, including many gem-based themes, that are automatically installed with it. To use one of these themes, simply specify it in the `_config.yml` file using the `theme` setting.

                ```yaml
                # filename: _config.yml

                # ...
                theme: jekyll-theme-slate   # Bundled with "gem github-pages".
                #theme: minima              # Default for "$ jekyll new".
                #theme: jekyll-theme-primer # Default for "gem github-pages".
                # ...
                ```

                * To see a list of the gem-based themes bundled with `github-pages`, run `$ bundle install` to install it, then run `$ bundle list` to view the installed gems. Look for entries that follow the `jekyll-theme-{name}` naming convention. One entry, `minima`, does not follow this naming convention. You need to know what to look for to check if it's installed.

                * Alternatively, the full list of `github-pages` gem dependencies on [https://pages.github.com/versions.json](https://pages.github.com/versions.json) will also show any bundled themes.

            * **Install and specify a new gem-based theme.** Install a theme by adding it to the project's `Gemfile` file, and specify it in the `_config.yml` file using the `theme` setting so Jekyll uses it when building the site.

                ```ruby
                # filename: Gemfile

                # ...
                gem "foobar"
                # ...
                ```

                ```yaml
                # filename: _config.yml

                # ...
                theme: foobar
                # ...
                ```

            * **Use a theme hosted on GitHub.** Reference a theme that is hosted on GitHub by specifying it in the `_config.yml` file with the `remote_theme` setting. This option tells Jekyll to pull the theme files from the remote repository when building the site, without having to download or install the template files locally.

                ```yaml
                # filename: _config.yml

                # ...
                remote_theme: account-name/repo-name@v3.2.1
                # ...
                ```

                * Format: `remote_theme: {account_name}/{repository_name}[@{version}]`.

                * This option works because the `github-pages` gem includes the `jekyll-remote-theme` plugin, which handles the fetching of all remote template files at build-time.

    * I wanted the first version of this site to have a simple design with very little styling that I could tweak and augment over time. The idea was to start off with a gem-based theme and write theme overrides to add new features; building a bespoke local theme was not appealing given all the available themes out there.

        I discovered a rich ecosystem of Jekyll themes, and since trying out different themes is easy (see above), I was able to quickly try out different ones. I eventually picked the Jekyll theme "Primer" theme to start - an official, minimalist theme with a bare-bones UX design, developed by GitHub for Jekyll sites.

        Jekyll theme sites:

        * Themes bundled with `github-pages`: [https://github.com/pages-themes](https://github.com/pages-themes)
        * GitHub repos Topic-tagged with `#jekyll-theme`: [https://jekyllrb.com/docs/themes/](https://github.com/topics/jekyll-theme)
        * Various community galleries: [https://jekyll.github.io/directory/themes](https://jekyll.github.io/directory/themes)
        * Popular GitHub Pages: [https://github.com/collections/github-pages-examples](https://github.com/collections/github-pages-examples)
        * Directory of Jekyll themes: [https://jekyll-themes.com/](https://jekyll-themes.com/)
        * Git docs: [https://github.com/git/git-scm.com](https://github.com/git/git-scm.com), [https://github.com/git/git.github.io](https://github.com/git/git.github.io)
        * Caught my eye:
            * [https://github.com/amzn/jekyll-doc-project](https://github.com/amzn/jekyll-doc-project)
            * [https://github.com/square/square.github.io](https://github.com/square/square.github.io)
            * [https://github.com/just-the-docs/just-the-docs](https://github.com/just-the-docs/just-the-docs)

    * BUG: Annoying bug when setting up a public repo with a LICENSE file using the Primer theme. The Primer theme uses the Liquid tag `{%raw%}{% github_edit_link ... %}{%endraw%}` in the `_layouts/default.html` template. That tag is triggered on public repos with a LICENSE file using the Primer theme, and causes the local build to break. STEPS TO REPRO: Create a repo with [visibility = Public], [theme = Primer], and a LICENSE file, and build the site locally. OBSERVED RESULTS: A `Liquid Exception: '@Jekyll::GitHubMetadata::EditLinkTag#parts' ...` error. EXPECTED RESULTS: A clean build. ROOT CAUSE: We believe the root cause is that it attempts to generate an "edit" link that points to the `gh-pages` branch by default, and since we decided to use the `main` branch for publishing, and no `gh-pages` branch was configured, executing this tag will break the build in our setup. STATUS: This is a [known issue of the `github-metadata` plugin](https://github.com/jekyll/github-metadata/issues/264). WORKAROUNDS: 1) Don't use a global LICENSE file; 2) override the `_layouts/default.html` template file to delete that flag; 3) override the `_layouts/default.html` template file to [build your own link](https://github.com/jekyll/github-metadata/blob/main/docs/edit-on-github-link.md); 4) investigate whether deploying from the `gh-pages` branch is indeed the root cause and do that.

    </details>

1. Export the Jekyll theme "Primer" to a local folder, to use as reference for files to override.

    ```shell
    #!/bin/zsh

    # Script settings.
    readonly _THEME_REPO="https://github.com/pages-themes/primer/"
    readonly _THEME_REPO_DTSLUG="${${${_THEME_REPO:t3}// /}//\//--}--$(date +%Y%m%dT%H%M%S)"

    # Export the Jekyll theme "Primer" to a local folder, to use as reference for template files.
    git clone --depth=1 "${_THEME_REPO}" "${_TMP_DIR}/${_THEME_REPO_DTSLUG}"
    rm -rf ./"${_TMP_DIR}/${_THEME_REPO_DTSLUG}"/.git
    ```

    <details markdown="1">
    <summary>More Info</summary>

    * Exporting a theme's files locally gives us a complete reference to all template files, making it easy to identify which specific files to create overrides for and customize the site. Follow this same approach to download other themes and add new features using templates and design elements from multiple sources.

    * Since we only want the theme's template files as reference, we don't need any Git metadata. To export a repo, we first create a shallow clone (`$ git clone --depth=1 ...`) to fetch as little metadata as possible, then we remove what little was downloaded (`$ rm .../.git`).

        * The temp directory we export to is `_tmp/`. The name is prefixed with `_` to ensure Jekyll ignores it - [Jekyll ignores folders that start with `_`](https://jekyllrb.com/docs/structure/) unless explicitly included in the `_config.yml` file. This temp directory is also in the `.gitignore` file, reducing clutter in the project repository.

        * The specific directory the theme is exported to is just a datetime-stamped slug of the repo we're downloading the theme from (e.g., `github.com--pages-themes--primer--19950624T181853`). The datetime stamp ensures every export is unique, in case the source repo changes between exports. We found ourselves downloading many different themes and this helped keep our local reference library of themes organized.

    </details>

1. Create a `_posts/` directory in the project root and create at least one post file therein.

    ```shell
    #!/bin/zsh

    # Create a `_posts/` directory for all blog posts.
    mkdir "_posts"

    # Create an initial stub post.
    cat <<CONTENT > _posts/1993-09-26--my-first-post.md
    ---
    layout: post
    title: My 1st Post
    ---
    This is my first post.
    CONTENT
    ```

    <details markdown="1">
    <summary>More Info</summary>

    * The resulting file should look like this:

        ```yaml
        # filename: _posts/1993-09-26--my-first-post.md

        ---
        layout: post
        title: My 1st Post
        ---
        This is my first post.
        ```

    * Posts are written in Markdown or HTML, they reside in the `_posts/` directory, and they require the filename format `YYYY-MM-DD-{title}.md` for Jekyll to process them.

        * The `{title}` can be whatever you want - Jekyll will slugify it to create a valid URL for the post (i.e., `my-first-post`), and titleize it to make it available for rendering (i.e., "My First Post").

        * For clarity, I use a double-dash delimiter, `--`, between the date and title parts of the filename. Jekyll handles it correctly.

    * The block at the top of the file delineated by three-dashed lines `---` is called a YAML front matter block. It contains a snippet of YAML that sets variables for the page used by the build engine. If a file has front matter then it is processed by Jeckyll at build-time and Liquid objects, tags, and filters can be used on the page. Without a front matter block, the file is not processed and is simply copied into the resulting static site as is.

    * The `layout: post` page variable tells Jekyll to use the `_layouts/post.html` template for this page.

    * A `title: My 1st Post` page variable will override the title Jekyll titelizes off the filename slug ("My First Post" in this example).

    </details>

1. Create an `index.html` file in the project root to serve as homepage and list all posts.

    ```shell
    #!/bin/zsh

    # Create an index file that simply lists all posts.
    cat <<CONTENT > index.html
    ---
    layout: default
    ---
    <ul>
        {%raw%}{% for post in site.posts %}{%endraw%}
        <li>
            [{%raw%}{{ post.date | date: "%Y-%m-%d" }}{%endraw%}]
            <a href="{%raw%}{{ post.url | relative_url }}{%endraw%}">{%raw%}{{ post.title }}{%endraw%}</a>
        </li>
        {%raw%}{% endfor %}{%endraw%}
    </ul>
    CONTENT
    ```

    <details markdown="1">
    <summary>More Info</summary>

    * The resulting file should look like this:

        ```liquid
        # filename: index.html

        ---
        layout: default
        ---
        <ul>
            {%raw%}{% for post in site.posts %}{%endraw%}
            <li>
                [{%raw%}{{ post.date | date: "%Y-%m-%d" }}{%endraw%}]
                <a href="{%raw%}{{ post.url | relative_url }}{%endraw%}">{%raw%}{{ post.title }}{%endraw%}</a>
            </li>
            {%raw%}{% endfor %}{%endraw%}
        </ul>
        ```

    * The `layout: default` page variable tells Jekyll to use the `_layouts/default.html` template for this page.

    </details>

1. Build the site locally and preview it to verify that it works.

    ```shell
    #!/bin/zsh

    # Build the site locally to verify that it works.
    bundle install                  # Install all project gems.
    bundle exec jekyll serve &      # Run a Jekyll server in the background.

    # View the local site.
    open "http://localhost:4000/"

    # Stop the background Jekyll server to clean up.
    kill %"bundle exec jekyll serve"
    ```

    <details markdown="1">
    <summary>More Info</summary>

    * The command `$ bundle install` installs all project gems.

        * This command installs all the gems required for the project: First, it reads a project's `Gemfile` to determine which gems are needed; next, it resolves all the dependencies and versions for all the listed gems; then, it fetches all the needed gems and installs them on the system; finally, it creates a `Gemfile.lock` file with the names and versions of the gems required by the project, ensuring the project has a consistent gem environment.

        * From this point on, best practice is to prefix any `jekyll` command with `bundle exec` (e.g., `$ bundle exec jekyll serve`). This ensures that Jekyll is using the exact versions of all project gems as specified in the `Gemfile.lock` file, ensuring a consistent gem environment.

    * The command `$ bundle exec jekyll serve &` builds and runs a local Jekyll server in the background.

        * If the build is successful, the command will output the local URL [http://localhost:4000/](http://localhost:4000/).

        * Append the ampersand character `&` to a command to run it as a background process. Running the Jekyll server in the background does not block access to the shell, allowing us to continue interacting with the terminal window and run other commands while the server is still running.

    * The command `$ kill %"bundle exec jekyll serve"` terminates the Jekyll server running in the background.

    * Useful shell commands:

        ```shell
        $ gem environment           # Display info on the project's gem environment.
        $ gem list                  # List of all gems installed in the system.
        $ gem list --details        # Detailed information on installed gems.
        $ bundle show               # List of names and versions of all project gems.
        $ bundle show --paths       # List of paths to all project gems.
        $ bundle install            # Install all project gems.
        $ bundle exec <cmd>         # Run <cmd> using gems in the `Gemfile.lock` file.
        $ bundle exec jekyll serve  # Build and serve the site locally.
        ```

    </details>

1. Stage all changes, commit them locally, and push them to GitHub.

    ```shell
    #!/bin/zsh

    # Stage all changes, commit them locally, and push them to GitHub.
    git add .                                   # Stage all files.
    git commit -m "Initial commit from local"   # Commit changes to local repo.
    git push origin                             # Push changes to remote repo.
    ```

1. Configure GitHub Pages for this repo.

    ```shell
    #!/bin/zsh

    # Configure GitHub Pages for this repo:
    echo "Configure GitHub Pages for this repo:"
    echo "  1. Go to the GitHub Pages Settings for this repo:"
    echo "     https://github.com/${_USERNAME}/${_REPO_NAME}/settings/pages"
    echo "  2. Select [ Source = \"Deploy from a branch\" ]."
    echo "  3. Select [ Branch > Select branch = \"main\" ]."
    echo "  4. Select [ Branch > Select folder = \"/ (root)\" ]."
    echo "  5. Click on \"Save\"."
    open "https://github.com/${_USERNAME}/${_REPO_NAME}/settings/pages"
    ```

    <details markdown="1">
    <summary>More Info</summary>

    * Online docs: [ [GitHub Pages docs](https://docs.github.com/en/pages) > [Configuring publishing source](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) > [Publishing from a branch](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-from-a-branch) ].

    * After a repo is updated, it takes a few minutes for the build process and publishing to complete.

    </details>

1. View the published site.

    ```shell
    #!/bin/zsh

    # Script settings.
    if [[ "${_REPO_NAME}" == "${_USERNAME}.github.io" ]]; then
        readonly _REPO_HOMEPAGE="https://${_REPO_NAME}/"                         # User site.
    else
        readonly _REPO_HOMEPAGE="https://${_USERNAME}.github.io/${_REPO_NAME}/"  # Project site.
    fi

    # View the published site.
    open "${_REPO_HOMEPAGE}"
    ```

    <details markdown="1">
    <summary>More Info</summary>

    * The exact URL will depend on the script settings. If no configuration variables were changed, it will be a GitHub Pages user site at `https://{username}.github.io`.

    </details>

That's it. You now have created a very simple GitHub Pages site.

There are a ton of things you can do next. In addition to creating more content (posts), you can also change the UX of the site by trying out different templates, or adding new features to yours via plugins or custom code. All of these changes are straight-forward to implement. The [Jekyll docs](https://jekyllrb.com/) and the [GitHub Pages docs](https://docs.github.com/en/pages) are good places to start.

Good luck!

## What About AI Coding Tools?

There are currently many AI coding tools out there: ChatGPT, Gemini, Claude Code, GitHub Copilot, etc.

I vibe-coded a working Jekyll site for GitHub Pages as one of my experiments. This took longer than expected and required a lot of hand-holding. Many of the build errors were resolved by crafting prompts to clarify intent, narrow scope, and guide output. To craft more effective prompts I found myself reading the GitHub Pages and Jekyll docs. By the time I was done with these docs I had a working site and most of this runbook developed.

Here is what I learned from this exercise:
* **AI tools provide great speed improvements.** AI coding assistants make writing new code faster by orders of magnitude. Faster code, however, does not always mean good code.
* **Effective guidance is critical.** While these tools are good at generating code, they still need clear, well-articulated, goal-oriented instructions to produce quality results. The hype over prompt engineering is not overblown: you need a solid understanding your objective to guide the tool to the desired outcome.
* **They are a complement, not a replacement.** These tools are most valuable for enhancing your ability to write, debug, and document code. But they are not a substitute for good planning - effective use still requires a solid understanding of architecture, design, and requirements to lead them effectively.

For this project, I expect these tools will be most useful in writing HTML/CSS/JavaScript code to update the site's templates, and in integrating and configuring feature plugins.