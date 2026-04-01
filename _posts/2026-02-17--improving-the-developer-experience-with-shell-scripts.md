---
title: Improving the Developer Experience with Shell Scripts
last_updated: 2026-04-01
---
# {{ page.title }}

<span style="color: #999">
_Published: {{ page.date | date: "%Y-%m-%d" }}_
&nbsp;&bull;&nbsp;
_Last Updated: {{ page.last_updated | date: "%Y-%m-%d" }}_
</span>

<span style="color: #999">
_Update (2026-04-01): Updated the application script template to make greater use of framework utility functions, revised the style guide for LLM-driven workflows, and made minor tweaks and edits._
</span>

I am a big fan of Developer Experience (DevEx), and I believe in workflow optimization as a way of protecting my focus. I'd rather spend my time and mental energy on high-level challenges than on repetitive minutiae, so I follow a simple rule: if a task repeats frequently enough, write a script.

For terminal-based tasks, shell scripting provides an environment to orchestrate command-line utilities with consistent and predictable logic. Shell scripts optimize your workflows by offloading the cognitive load of minor details. In practice, they:

* Abstract the complex implementation details of any task behind a single, descriptive command;

* Save you from having to re-learn repetitive tasks; and

* Codify complex processes into living documentation for future reference.

I manage my personal shell scripts using a simple architecture:

1. **Central Script Location.** All my personal scripts reside in `~/_local/bin/` and `~/_local/lib/`. This follows my overall standards for managing my personal code and data.

1. **Standard Application Script Template.** Every application script is based on a boilerplate template. This ensures that every new script starts off with a consistent structure, error handling, and logging - I don't have to start new scripts from scratch!

1. **Common Script Framework.** Every script sources a central framework initialization script to ensure a standardized execution environment. This bespoke framework sets up a unified environment - a consistent baseline for shell settings, common utility functions, and global variables. This simplifies the writing and maintenance of scripts.

1. **Pragmatic Style Guide.** Every scripts follows a simple style guide for naming conventions, code formatting, and design patterns. This makes it easier to reuse code across scripts, and to maintain a growing library of scripts.

In this post, I cover these points in more detail.

## Shell Scripts and Workflow Optimization

While shell scripts are designed to automate tasks and reduce the likelihood of human error, their true value lies in optimizing your workflow so you can reclaim mental bandwidth to focus on solving big-picture problems.

Consider, for example, initializing a new remote repo. While the task appears straightforward, the steps and effort to complete it depend on context: does your IDE have the extensions to do it via a GUI? Are you starting from scratch or from a local project? Is your local project already under version control or not? Is there a naming conflict with a remote repo? Do you remember the commands to use? A script can handle this logic and ensure the right commands are executed for each scenario, saving me the time to look up and execute every command.

I context-switch all the time - across projects, across technology stacks, and across mental models. Because I'm not performing every task daily, I often find myself having a "wait, how do I do this again?" moment. Scripts capture the expertise for performing tasks so I don't have to re-learn how to do them over and over again. They save me the time and effort of figuring things out, and they reduce my cognitive load. By encapsulating multi-step workflows under single commands, I create an abstraction layer so I only have to remember the name of the script for the task I need.

Beyond daily tasks, scripts also serve as living documentation. For example, I did not write [a script for setting up this blog]({% post_url 2025-08-07--setting-up-this-blog-on-github-pages %}) because I see myself creating blogs all over the place. I wrote it to codify a complex process. It is a reference for reusable patterns and, ultimately, [a guide to my future self]({% post_url 2025-08-01--about-this-site %}). Any time I revisit a script in the future, I know I am starting from a solution that works.

I have curated a small library of scripts over time and I now manage them holistically and systematically as part of my global setup. The rest of this post describes how I manage my personal shell scripts.

## Script Location

All my personal scripts are centralized under the `~/_local/` folder.

* Scripts are under a `_local/` directory.

    * This mirrors [the way I organize my source code]({% post_url 2025-12-29--how-i-organize-source-code-on-my-computer %}). The name "_local" with an underscore prefix indicates a self-managed space that is scoped, centralized, and easy to find. This structure also ensures portability: migrating to a new machine is as simple as copying a folder or running a setup script.

    * `_local/(bin,lib,...)/` aligns with the nomenclature used by macOS and Linux, and disambiguates from other locations used for tools and scripts. This is a dedicated directory for my tools and configs. I am simply adding a location that I automatically recognize as one I manage.

        Why this new location?  On macOS, for example, scripts reside in various locations:

        | Path | Owner | Purpose |
        | :--- | :--- | :--- |
        | **`/bin/`** | Core OS | Tools required to boot or repair the system. |
        | **`/usr/bin/`** | Package Manager | Tools managed by the OS package manager. |
        | **`/usr/local/bin/`** | Sys Admin | Third-party software installed for all users. |
        | **`~/.local/bin/`** | User | Personal executables and scripts. |

        Even though `~/.local/bin/` is touted as the location for personal tools and scripts, in practice it is a bit of a dumping ground for various CLI installers (e.g., GitHub CLI, Ruby Gems, etc.) and other tools. Using `~/.local/bin/` would make it cumbersome to pick out my personal scripts. A new directory avoids bloat.

* The `_local/` directory is in the root of the home folder `~/`.

    * This gives me a common location to set up and work from on any machine I use.

        ```sh
        ~/_local/
        ├── bin/        # Executable scripts.
        ├── lib/        # Script libraries.
        └── ...
        ```

* My shell's `PATH` variable prepends `~/_local/bin/` to prioritize my scripts over others.

    * On macOS, for example, my Z shell config file includes the following entries:

        ```zsh
        # filename: ~/.zshrc

        # Prepend personal scripts to PATH, to prioritize over system defaults.
        export PATH="${HOME}/_local/bin:${PATH}"
        ```

## Application Script Template

Application scripts in my `~/_local/bin/` folder generally follow a common template. Using a consistent structure reduces my cognitive load when opening scripts months later, when creating new ones, or scavenging for code patterns across other scripts in my library.

The template manages environment initialization, implements default argument parsing, and provides markers for app-specific implementation (i.e., all `TODO` entries), so working on a new app feels more like a "fill-in-the-blank" form.

Below is a snapshot of my application script template `~/_local/bin/script-template.zsh`.

```sh
#!/usr/bin/env zsh
# -----------------------------------------------------------------------------
# SPDX-FileCopyrightText:   (c) 2025 Mauricio Lomelin <maulomelin@gmail.com>
# SPDX-License-Identifier:  MIT
# SPDX-FileComment:         Application Script
# SPDX-FileComment: <text>
#   This is the base template for Zsh shell scripts.
#   Configure the script by addressing all "TODO" tasks.
#   TODO: Delete this "SPDX-FileComment" block and all "TODO" tasks before use.
# </text>
# -----------------------------------------------------------------------------

# Initialize script framework (use `dirname` and `printf` for portability).
source "$(dirname "${0}")/../lib/framework/init.zsh" || {
    printf "\e[91mError: Failed to initialize script framework.\e[0m\n"
    exit 1
}

# Prevent script sourcing.
if [[ ${ZSH_EVAL_CONTEXT} == *:file* ]]; then
    log::warning "The script [${(%):-%x}] must be executed, not sourced."
    return 1
fi

# Initialize private registry.
typeset -gA _APP=(
    [DEFAULT_VERBOSITY]=${REG[DEFAULT_VERBOSITY]}
    [DEFAULT_BATCH]=${REG[DEFAULT_BATCH]}
    [DEFAULT_HELP]=${REG[DEFAULT_HELP]}
    # TODO: Define additional app-specific constants and settings here.
)

# Display help documentation and exit. Invoked as needed.
function usage() {
# ----------------- 79-character ruler to align usage() text ------------------
    cat << EOF
Usage:

    # TODO: Add additional parameters/flags to the command interface below.
    ${ZSH_ARGZERO:A:t} [-v=<level>] [-b] [-h]

Description:

    # TODO: Write a brief description of the script here.

Options:

    # TODO: Document additional script parameters/flags here.

    -v=<level>, --verbosity=<level>
        Sets the display threshold for logging level.
        Defaults to [${_APP[DEFAULT_VERBOSITY]}] if not present or invalid.

            Log Message    |   Verbosity Level
              Display      |  0   1   2   3   4
        -------------------+--------------------
                0/Alert    |  Y   Y   Y   Y   Y
          Log   1/Error    |  N   Y   Y   Y   Y
         Level  2/Warning  |  N   N   Y   Y   Y
                3/Info     |  N   N   N   Y   Y
                4/Debug    |  N   N   N   N   Y

    -b, --batch
        Force non-interactive mode to perform actions without confirmation.

    -h, --help
        Display this help message and exit.
EOF
    exit 0
}

# Implement core logic. Invoked by main().
function run() {

    # Map function arguments to local variables.
    # TODO: Map additional function arguments to local variables here.
    local batch="${1}"

    # TODO: Implement script's core logic here.
    log::info_header "log::info_header()"
    log::info "log::info()"
    log::debug "log::debug()"
    log::warning "log::warning()"
    log::error "log::error()"
    log::alert "log::alert()"
}

# Parse and validate CLI arguments. This is the script's entry point.
function main() {

    # Parse all CLI arguments.
    # TODO: Declare local variables for additional parameters/flags here.
    local help batch verbosity
    local -a args=( "${@}" ) args_used=() args_ignored=()
    while (( $# )); do
        case "$1" in
            (-h|--help)          help=true           ; args_used+=(${1}) ;;
            (-b|--batch)         batch=true          ; args_used+=(${1}) ;;
            (-v=*|--verbosity=*) verbosity="${1#*=}" ; args_used+=(${1}) ;;
            # TODO: Parse additional parameters/flags here.
            (*)                                        args_ignored+=(${1}) ;;
        esac
        shift
    done

    # Set verbosity level.
    log::set_verbosity "${_APP[DEFAULT_VERBOSITY]}" # Set level to app default.
    log::set_verbosity "${verbosity}"               # Try to set to user input.
    verbosity=$(log::get_verbosity)                 # Get actual level.

    # Log script identifier to mark the start of all logging.
    log::info_header "# TODO: Give the script a short, friendly name here."

    # Handle help requests before validating other inputs.
    help=$(dat::validate_bool "help flag" "${help}" "${_APP[DEFAULT_HELP]}") || return 1
    if dat::is_true "${help}"; then usage; fi

    # Validate all other inputs.
    batch=$(dat::validate_bool "batch flag" "${batch}" "${_APP[DEFAULT_BATCH]}") || return 1
    # TODO: Validate/initialize additional parameters/flags here.

    # TODO: Perform input checks that would result in a sys::abort() here.

    # Display all processed arguments.
    log::info "Arguments processed:"
    log::info "  Input:        [${args}]"
    log::info "  Used:         [${args_used}]"
    log::info "  Ignored:      [${args_ignored}]"
    log::info "Default settings:"
    # TODO: Include default settings for additional parameters/flags here.
    log::info "  Verbosity:    [${_APP[DEFAULT_VERBOSITY]}]"
    log::info "  Batch:        [${_APP[DEFAULT_BATCH]}]"
    log::info "  Help:         [${_APP[DEFAULT_HELP]}]"
    log::info "Effective settings:"
    # TODO: Include values for additional parameters/flags here.
    log::info "  Verbosity:    [${verbosity}]"
    log::info "  Batch:        [${batch}]"
    log::info "  Help:         [${help}]"

    # TODO: Perform input checks that would result in a log::warning() here.

    # Prompt user for confirmation, unless in batch mode.
    if dat::is_true "${batch}"; then
        log::warning "Batch mode enabled. Proceeding with script."
    else
        read "response?Proceed? (y/N): "
        if ! dat::is_yes "${response}"; then
            sys::terminate "User declined to proceed."
        fi
    fi

    # Check that all variables are populated before executing core logic.
    # TODO: Add all run() arguments to the array below.
    local -a args=( "${batch}" )
    if [[ "${#args}" != "${#args:#}" ]]; then
        sys::abort "Invalid state: Check args."
    fi

    # Execute core logic.
    run "${args[@]}"
}

# Invoke main() with all CLI arguments.
main "${@}"
```

<details markdown="1">
<summary>Personal Notes</summary>

* To initialize the shell script framework, source the library `$(dirname "${0}")/../lib/framework/init.zsh`.

    * Although I prefer to avoid using external commands, this first line is written in more portable form (i.e., POSIX compliant) to ensure I always get to the initialization library where I run more thorough checks.

    * Since my personal scripts and libraries are always in `_local/bin/` and `_local/lib/`, respectively, the path to source the environment loader from a script is always `../lib/framework/init.zsh`.

    * If portability was not a concern for this command, I would use zsh-native ways to get the script's directory path instead of `$(dirname "${0}")`:

        * The history expansion modifier `${0:A:h}` could be used. It uses shell expansion on `${0}`, without relying on an external command.

            * `${0}` usually represents the name of the script being executed.

            * `A` is a history expansion modifier that turns a filename into an absolute path and resolves symbolic links (<a href="https://zsh.sourceforge.io/Doc/zsh_us.pdf#page=59" target="_blank">The Z Shell Manual v5.9 § 14.1.4</a>).

            * `h` is a history expansion modifier that removes a trailing pathname component. It works like `dirname` (<a href="https://zsh.sourceforge.io/Doc/zsh_us.pdf#page=59" target="_blank">The Z Shell Manual v5.9 § 14.1.4</a>).

        * The modifier `${${(%):-%x}:A:h}` is a more robust, however, because there are edge cases where `${0}` does not contain the script's name. In the expression `${(%):-%x}`:

            * The `${:-word}` parameter expansion operator always returns `word` (<a href="https://zsh.sourceforge.io/Doc/zsh_us.pdf#page=64" target="_blank">The Z Shell Manual v5.9 § 14.3</a>).

            * `(%)` is a parameter expansion flag that expands all `%` escapes (<a href="https://zsh.sourceforge.io/Doc/zsh_us.pdf#page=69" target="_blank">The Z Shell Manual v5.9 § 14.3.1</a>).

            * The `%x` word is the name of the file containing the source code currently being executed (<a href="https://zsh.sourceforge.io/Doc/zsh_us.pdf#page=52" target="_blank">The Z Shell Manual v5.9 § 13.2.3</a>).

* To define private global variables used by a module, we use the global associative array `typeset -gA _APP=( ... )`. This map is then configured with every variable or constant the script will use. The framework defines many constants in a global public registry `typeset -grA REG=( ... )` that can be used to initialize module variables, as shown.

* In the `usage()` function use `${ZSH_ARGZERO:A:t}` to get the script's name shown because:

    * The easier-looking `$(basename "${ZSH_ARGZERO}")` uses an external command. I prefer to avoid those.

    * `A` is a history expansion modifier that turns a filename into an absolute path and resolves symbolic links (<a href="https://zsh.sourceforge.io/Doc/zsh_us.pdf#page=59" target="_blank">The Z Shell Manual v5.9 § 14.1.4</a>).

    * `t` is a history expansion modifier that removes all leading pathname components, leaving the tail. It works like `basename` (<a href="https://zsh.sourceforge.io/Doc/zsh_us.pdf#page=60" target="_blank">The Z Shell Manual v5.9 § 14.1.4</a>).

* In the CLI argument parser, the parameter expansion `${1#*=}` removes everything up to the first "=" sign from the parameter `${1}`. E.g., if `${1}` was `foobar=fizzbuzz`, then `${1#*=}` would be `fizzbuzz`.

    * The `${name#pattern}` parameter expansion pattern (<a href="https://zsh.sourceforge.io/Doc/zsh_us.pdf#page=65" target="_blank">The Z Shell Manual v5.9 § 14.3</a>) using the `*=` glob operator as the pattern (<a href="https://zsh.sourceforge.io/Doc/zsh_us.pdf#page=84" target="_blank">The Z Shell Manual v5.9 § 14.8.1</a>), extracts the value from a `key=value` string.

    * The choice to use "=" as parameter/value delimiter for any parameter passed to the function was a personal choice. It was valuable for me to implement simple parsing logic that is easy to maintain. The same model applies to both short-form and long-form options/flags. The following two commands are equivalent:

        ```sh
        $ script -h -v=3 -b
        $ script --help --verbosity=3 --batch
        ```

* The option descriptions in `usage()` are displayed below the parameters/flags with a hanging indent.

    * I found this to be the best layout for a 79-character width guide.

        ```sh
        # ----------------- 79-character ruler to align usage() text ------------------
        Options:

            # Rendering in columns has issues with long parameter names:
            -b, --batch                                         Descriptions are hard
                                                                to read in narrow cols.

            -l=<level>, --a_very_long_parameter_name=<level>    Difficult to write
                                                                structured text here.

            # Stacking flags & parameters has long name issues and can be hard to read:
            -b                                     Force non-interactive mode.(AND/OR?)
            --batch                                Perform actions w/out confirmation.

            -l=<level>                             This description still has a narrow
            --a_very_long_parameter_name=<level>   column to render.

            # Hanging indents make better use of horizontal and vertical space:
            -h, --help
                This description can make use of most of the available width.

            -l=<level>, --a_very_long_parameter_name=<level>
                This layout allows long names and has a very wide description field.
        ```

* Utility functions that look like this `log::info()`, `log::get_verbosity()`, `dat::is_true()` and others in the template are part of the framework. The framework provides many utility services that create abstractions to help keep you focused on the script logic instead of the implementation detail.

    * For example, any of the strings `true`, `on`, `yes`, and `1` can be used for the value `true` and any of the strings `false`, `off`, `no`, and `0` for the value `false`. The utility function `dat::is_true()` handles all of this, so you don't have to worry about which one is used in a variable. This keeps you focused on the logic and not on the implementation details.

    * The framework provides utility services for logging, error handling, type validation, config management, and other areas. They live in `~/_local/lib/framework/` but are beyond the scope of this post.

</details>

## Framework Initialization Script

All application scripts begin by sourcing a framework initialization script that configures the environment and sources all framework libraries. It's the line in the template that starts with `source "$(dirname "${0}")/../lib/framework/init.zsh"`.

The initialization script ensures the environment runs under zsh, configures error handling options, and validates the namespaced architecture by checking for function name collisions before sourcing all framework script libraries.

Below is a snapshot of my initialization script `~/_local/lib/framework/init.zsh`.

```sh
#!/usr/bin/env zsh
# -----------------------------------------------------------------------------
# SPDX-FileCopyrightText:   (c) 2025 Mauricio Lomelin <maulomelin@gmail.com>
# SPDX-License-Identifier:  MIT
# SPDX-FileComment:         Framework Initialization Script
# -----------------------------------------------------------------------------

# Fail fast if not running under zsh.
#   - ${ZSH_NAME} is only set if the script is running under zsh.
#   - Use single brackets and `printf` for portability.
if [ -z "${ZSH_NAME}" ] ; then
    printf "\e[91m"
    printf "Error: This script requires zsh.\n"
    printf "\n"
    printf "  - To run it in a zsh shell, either:\n"
    printf -- "      - Invoke it with zsh:  \`$ zsh script.zsh\`\n"
    printf -- "      - Execute it directly: \`$ chmod +x script.zsh ; ./script.zsh\`\n"
    printf "\n"
    printf "  - To use a different shell, modify scripts accordingly.\n"
    printf "\n"
    printf "\e[0m"
    return 1
fi

# Prevent direct execution.
if [[ ${ZSH_EVAL_CONTEXT} != *:file* ]]; then
    echo "\033[91mError: This script must be sourced, not executed.\033[0m"
    return 1
fi

# Enable strict error handling and debugging.
#   - Use full option names ("The Z Shell Manual" v5.9, § 16, pg. 111).
setopt ERR_EXIT        # Exit on errors.
setopt NO_UNSET        # Exit on undefined variables.
setopt PIPE_FAIL       # Fail if any command in a pipeline fails.
setopt TYPESET_SILENT  # Silence variable re-declarations.
setopt NO_CASE_MATCH   # Enable case-insensitive pattern matching.
setopt EXTENDED_GLOB   # Enable extended globbing features.
#setopt XTRACE         # DEBUG: Trace command execution and expansion.

# Source framework libraries.
#   - Source framework libraries only if no function name collisions are found.
#   - Run checks inside an anonymous function to keep the global scope clean.
function () {

    # Framework libraries.
    local lib_dirpath="${${(%):-%x}:A:h}/lib"
    local -a libs=(
        "reg--global-registry.zsh"
        "log--logging.zsh"
        "sys--system.zsh"
        "dat--data-types.zsh"
        "env--environment-info.zsh"
        "err--error-handling.zsh"
        "ded--graveyard.zsh"
        "cfg--config-mgmt.zsh"
        # TODO: Add new libraries here.
    )

    # -----------------------------------------------------------------------------
    # Syntax:   _extract_function_names_from_file <file>
    # Args:     <file>      A file name.
    # Outputs:  An array of function names found inside <file> using regexes.
    # Status:   Default status.
    # Notes:    This function extracts function names to check for name collisions.
    #           Locally scoped functions are likely intended to shadow an original.
    #           If we assume indented function definitions to be locally scoped and
    #           non-indented to be original, indented definitions escape detection.
    # -----------------------------------------------------------------------------
    function _extract_function_names_from_file() {
        local file=${1:-}
        # Define RegEx patterns to extract function names (fnames) from files.
        local re_pre="^function[[:space:]]+"                # Left of fname.
        local re_fn="[a-zA-Z0-9_:]+"                        # fname.
        local re_post="[[:space:]]*\(\)[[:space:]]+\{.*$"   # Right of fname.
        # Get an array of function names from the given file.
        local -a fnames=( ${(f)"$( grep -E "${re_pre}${re_fn}${re_post}" "${file}" | sed -E "s/${re_pre}// ; s/${re_post}//" )"} )
        echo "${fnames}"
    }

    # Create a function name registry.
    #   - Functional schema: fnmap[fn] => { (int)count, (str)sources }
    #   - Implement as parallel associative arrays managed by local functions.
    local -A fn_count   # The number of sources for a given fn.
    local -A fn_sources # The list of sources a given fn is found in.

    # -----------------------------------------------------------------------------
    # Syntax:   _fnmap_add_source_to_function_names <source> [<fname> ...]
    # Args:     <source>    A source string.
    #           <fname>     A list of function names.
    # Outputs:  None
    # Status:   Default status.
    # Details:
    #   - Appends <source> to the list of sources of each function name in the
    #     fnmap registry. If no function name index is found, one is added.
    #   - Updates the count of <source>s added to a function name.
    # -----------------------------------------------------------------------------
    function _fnmap_add_source_to_function_names() {
        local source=${1}           # Map source argument to local variable.
        local -a fns=( "${@:2}" )   # Map list of function names to an array.
        local fn
        for fn in ${fns}; do
            fn_count[${fn}]=$(( ${fn_count[${fn}]:-0} + 1 ))
            fn_sources[${fn}]=${fn_sources[${fn}]:-}${fn_sources[${fn}]:+, }${source}
        done
    }

    # -----------------------------------------------------------------------------
    # Syntax:   _fnmap_validate_fnames
    # Args:     None.
    # Outputs:  An error message for every duplicate in the function name registry.
    # Status:   returns 0 (success) if no duplicates found.
    #           returns 1 (error) if duplicates found.
    # -----------------------------------------------------------------------------
    function _fnmap_validate_fnames() {
        local duplicates=false
        local fn
        for fn in ${(k)fn_count}; do
            if (( fn_count[${fn}] > 1 )); then
                echo "\e[91mError: Function name duplicates detected:"
                echo "    Function:\t${fn}()"
                echo "    Sources:\t${fn_sources[${fn}]}"
                echo "==> Revise function names to avoid collisions."
                echo "\e[0m"
                duplicates=true
            fi
        done
        if [[ "${duplicates}" == true ]]; then
            return 1    # Exit function with an error.
        else
            return 0    # Exit function with status ok.
        fi
    }

    # -----------------------------------------------------------------------------
    # Syntax:   _fnmap_print
    # Args:     None.
    # Outputs:  Pretty-prints the function name registry to stdout, in JSON format:
    #           { "fnmap": { "<fname>": { "count": int, "sources": str } } }
    # Status:   Default status.
    # -----------------------------------------------------------------------------
    function _fnmap_print() {
        echo "{"
        echo "  \"fnmap\": {"
        local fn
        for fn in ${(k)fn_count}; do
            echo "    \"${fn}\": {"
            echo "      \"count\": ${fn_count[${fn}]},"
            echo "      \"sources\": \"${fn_sources[${fn}]}\""
            echo "    }"
        done
        echo "  }"
        echo "}"
    }

    # Process all libraries.
    local -a fns
    local lib
    for lib in ${libs[@]}; do
        fns=( $(_extract_function_names_from_file "${lib_dirpath}/${lib}") )
        _fnmap_add_source_to_function_names "${lib}" "${fns[@]}"
    done

    # Process the executable script.
    fns=( $(_extract_function_names_from_file "${ZSH_ARGZERO:A}") )
    _fnmap_add_source_to_function_names "${ZSH_ARGZERO:A:t}" "${fns[@]}"

    # Process the environment.
    fns=( ${(k)functions} )
    _fnmap_add_source_to_function_names "(environment)" "${fns[@]}"

    # DEBUG: Print function name registry.
    #_fnmap_print

    # Validate function names and exit function if collisions are detected.
    _fnmap_validate_fnames || return 1

    # Source common libraries if no collisions were detected.
    local lib
    for lib in "${libs[@]}" ; do
        source "${lib_dirpath}/${lib}"
    done

} || return 1   # Catch and return any errors to the caller.
```

<details markdown="1">
<summary>Personal Notes</summary>

* All of my library scripts are in the some folder relative to the initialization script. The modifier `${${(%):-%x}:A:h}` resolves into the absolute path of the initialization script. All library scripts are sourced relative to that.

    * The `${:-word}` parameter expansion operator always returns `word` (<a href="https://zsh.sourceforge.io/Doc/zsh_us.pdf#page=64" target="_blank">The Z Shell Manual v5.9 § 14.3</a>).

    * `(%)` is a parameter expansion flag that expands all `%` escapes (<a href="https://zsh.sourceforge.io/Doc/zsh_us.pdf#page=69" target="_blank">The Z Shell Manual v5.9 § 14.3.1</a>).

    * The `%x` word is the name of the file containing the source code currently being executed (<a href="https://zsh.sourceforge.io/Doc/zsh_us.pdf#page=52" target="_blank">The Z Shell Manual v5.9 § 13.2.3</a>).

* Anonymous functions have an exit status just like regular functions. However, since they are executed immediately, the exit status is the exit status of the entire expression. We use the following pattern to catch any errors immediately after the function block:

    ```sh
    function () {   # Anonymous functions execute immediately upon definition.
        # ...
        return 1    # Exit the anonymous function with an error.
        # ...
    } || return 1   # Catch and return any errors to the caller.
    ```

* The anonymous function block checks for function name conflicts across all sourced libraries, the application script, and the current environment. The style guide has a rule for function definitions so we can scan for them. This gives us some freedom in picking names because we know the initialization script will always check for name collisions.

</details>

## Style Guide

My scripts share a consistent look, feel, and flow because every script I write follows a set of rules that define a common design language. This is my personal style guide. It embraces defensive coding techniques and structural guidelines that make my code easier to write, read, and maintain.

The rules are straightforward and apply to both application and library scripts:

* **Favor readability over cleverness.**

    * When I go back to a script, a clever trick today will only waste my time deciphering it in the future, whereas readable code that is well commented lets me work without a steep learning curve.

* **Write for Z shell (zsh).**

    * I have lately favored writing for Z shell because it is the default interactive shell on macOS (I work mostly on macOS), and I find it easier to use over other shells.

    * I can reference a single document source for how the script operates: <a href="https://zsh.sourceforge.io/" target="_blank">The Z Shell Manual</a>).

    * I only use portable code (i.e., POSIX compliant) when absolutely necessary.

* **Use a `.zsh` extension for Z shell scripts and appropriate extension for other interpreters.**

    * Use appropriate file extensions for all shell scripts (e.g., `*.sh`, `*.bash`, `*.ksh`). This is a best practice for explicitly indicating the intended interpreter at the file level.

* **Use descriptive names on all script filenames.**

    * Scripts should be easy to recognize by looking at their filename, whether it is a command script or a library script. `lib/framework/lib/cfg--config-mgmt.zsh` or `bin/install-shell-scripts.zsh` are easy to recognize at a glance.

* **Start all script files with `#!/usr/bin/env zsh`.**

    * This is supposed to make scripts more portable, but I do this for explicit identification. I run scripts by invoking the zsh interpreter explicitly with `$ zsh script.zsh`, bypassing the shebang. The shebang is there for direct execution with `$ ./script.zsh` (requires `$ chmod +x script.zsh` first), and to clarify the interpreter when only looking at the code.

* **Favor Zsh builtins over external commands.**

    * Z shell builtins can be used to replace many external commands. I prefer Zsh builtins because they remove an external dependency from my scripts. For example:

        ```sh
        local fmt_datetime="%Y%m%dT%H%M%S"
        local datetime script

        # DO THIS:
        datetime="${(%):-"%D{${fmt_datetime}}"
        script="${ZSH_ARGZERO:A:t}"

        # DOABLE (but not preferred):
        datetime="$(date ${fmt_datetime})"
        script="$(basename "${ZSH_ARGZERO}")"
        ```

My scripts use a namespaced architecture. Since shell scripts share a global namespace, this is a defensive practice I leverage to reduce potential naming conflicts across scripts, to make library functions discoverable, and to reduce global namespace pollution by encapsulating all global variables. Here is what this entails:

* **Assign a unique 3-character namespace code to every framework utility library.**

    * The namespace code is a unique 3-character string assigned to each framework utility library (e.g., "LOG", "SYS", "DAT"). I capture it in the filename, in the file's comment header, and the code is used by other rules - they are a core part of how my framework is architected and managed.

* **Use `{namespace}--{description}.zsh` for all framework utility library filenames.**

    * Any script file under `~/_local/lib/framework/lib/`, such as `log--logging.zsh` or `sys--system.zsh`. This convention ensures namespaces are unique across all framework libraries.

* **Use `{namespace}::{fname}()` for all public function names.**

    * Make sure the namespace prefix is in lower-case (e.g., `log::info()`, `log::error()`). Given unique namespaces, this eliminates the likelihood of name collisions.

* **Define all functions starting with the regex pattern `^function [a-zA-Z0-9_:]+\(\) \{$`.**

    * Because shell scripts share a global namespace, the initialization script checks to make sure all function names are unique across all libraries, executable, and environment. Using this function prototype pattern as the first line of any function definition ensures it will be picked up.

    * Why do we anchor a function name to the beginning of a line (i.e., with `^function...`)? Locally scoped functions (i.e., those inside other functions) are likely intended to shadow an original function within that scope. If we assume that indented functions are locally scoped and non-indented functions are globally scoped, then we ignore the indented ones.

        ```sh
        function log::info_x() { ... }      # Global function - Check for collisions.

        function experiment() {
            function log::info_x() {        # Shadow function - Ignore for collisions.
                echo "${@}" >&2
            }
            log::info_x "shadow log"        # Uses shadow function.
        }
        ```

* **Use `_{fname}()` for all private script function names.**

    * The convention and only requirement is to use a leading underscore (e.g., `_to_multiline()`) for functions private to a module. Without any additional constraints, there is no guarantee that private function names are unique across all scripts. I could have enforced namespace prefixes on all private functions to make them unique (e.g., `_log::to_multiline()`). However, it turns out this makes scripts harder to read and maintain. Thus, I relaxed this requirement and instead added code in the initialization script to run a check on all files to make sure all function names are unique - readability over elaborate schemes.

* **Use the map `[_]{namespace}[]` as a module registry to hold global variables.**

    * I implement module registries using global associative arrays to hold all global variables used by a module. By convention I use `_{namespace}` for private entries, and `{namespace}` for public entries. This encapsulation adds only 2 variables per module to the global namespace, thus limiting global namespace pollution.

        ```sh
        # file: fbr--foobar.zsh

        typeset -gA _FBR=(      # Private registries are named "_{namespace}".
            [SETTING]=3         # Private constants are in upper-case.
            [state]="a"         # Private mutables are in lower-case.
        )

        typeset -gA FBR=(       # Public registries are named "{namespace}".
            [DEFAULT_STATE]="X" # Public constants are in upper-case.
            [variance]=0.1      # Public mutables are in lower-case.
        )
        ```

* **Use `{namespace}::(get|set)_{variable}()` getter/setter functions to access public variables across modules.**

    * For example, `log::get_verbosity()`, `log::set_verbosity()`. This abstraction ensures I can validate any new values, I can update other state variables as needed, and it decouples users of that API from needing to worry about the internal workings of a module - standard API stuff.

When writing the actual logic of a script, there are a few rules and common patterns that make them easier to maintain.

* **Write a header comment for all public functions in a library.**

    * Even if the function is obvious, this is a good habit to keep. This should be the only documentation a developer needs to read to learn how to use the library; they should not need to read code. This is an example of the header comment block I use:

        ```sh
        # -----------------------------------------------------------------------------
        # Syntax:   function_name <req> ... [<opt> ...]
        # Args:     <req>       Brief description of required arguments.
        #           <opt>       Brief description of optional arguments.
        # Outputs:  Describe the output of the function and some of the logic used to
        #           generate different outputs based on different conditions.
        # Status:   Describe any return and exit status codes.
        # Details:
        #   - Bulleted specification of the function, as needed.
        # -----------------------------------------------------------------------------
        function function_name() { ... }
        ```

* **Use explicit type declarations.**

    * Declare every variable used, and always use explicit flags to clarify the intent of the variable being declared. Not only does this serve as self-documenting code, but it makes it obvious when reading code what scope and underlying data structure you're dealing with.

        ```sh
        # Local scope.      # Global scope.
        local scalar    ;   typeset -g global_scalar
        local -a array  ;   typeset -ga global_array
        local -A map    ;   typeset -gA global_map
        ```

* **Keep declaration and assignment as separate statements.**

    * The `local` builtin does not propagate the exit code from a command substitution, so a declaration will shadow the exit status of the subshell. In the "DO NOT DO THIS" example below, the `return` short-circuit will never be triggered.

        ```sh
        # DO THIS:
        local x
        x=$(dat::as_bool ...) || return 1

        # DO NOT DO THIS:
        local x=$(dat::as_bool ...) || return 1
        ```

* **Narrow the scope of variable declarations as much as possible.**

    * Shell scripts share a global namespace. Because of this, limiting the scope of any variable to the block of code that needs it prevents namespace pollution, avoids name collisions, and makes it easier to read and maintain scripts.

    * If a variable is only used in a local scope, declare it locally with `local {variable}`:

        ```sh
        function foo() {    # Inside a function.
            local var       # Without the `local` keyword, "var" becomes global.
            # ...
        }

        function () {       # Inside an anonymous function.
            local var       # Without the `local` keyword, "var" becomes global.
            # ...
        }
        ```

    * If a private variable is used across functions, define it in the private registry and access it directly within the module:

        ```sh
        typeset -gA _FBR=(              # Private registry for module "Foobar (FBR)".
            _FBR[MIN_COUNT]=0           # Private constant.
            _FBR[count]=0               # Private variable/mutable.
        )

        function _increment_count() {   # Increment counter.
            (( _FBR[count]++ ))
        }
        function _decrement_count() {   # Decrement counter down to MIN_COUNT.
            (( _FBR[count] > _FBR[MIN_COUNT] )) && (( _FBR[count]-- ))
        }
        ```

        Writing getter/setter functions for accessing private variables within a module is overkill for my personal scripts.

    * Use anonymous functions to declare code that executes right away and clears out any locally-scoped variables when complete.

* **Always check return values and bubble up status codes.**

    * Use logical AND or check for errors manually.

        ```sh
        # DO THIS:
        cp source.txt target.txt && echo "Success"

        # DO THIS:
        cp source.txt target.txt || return $?
        echo "Success"
        ```

    * When short-circuiting a response, always bubble up any error status codes (i.e., `some_action || return 1` until you get to the top level. At the top level, one can then create traps to handle the error.

    * TODO: I still need to decide on a standard way to bubble up status codes:
        * Simplify things and always execute a `return 1`.
        * Be transparent and use the original code via `$?`.
        * Customize the meaning of an error by executing a custom `return {n}`.

* **When passing an array, always do it explicitly.**

    * Reference arrays explicitly via `"${foo[@]}"` or `"${(@)foo}"` (we prefer the former). Referencing an array with `"${foo}"` is equivalent to accessing a scalar value, which results in the first item. This will cause loops and passing arrays into functions to only treat the first element of the array. In double quotes, array elements are put into separate words, so that `"${foo[@]}"` is the same as `"${foo[1]}" "${foo[2]}" ...`.

        ```sh
        local -a foo=( "${@}" )

        # DO THIS:
        do_something "${foo[@]}"

        # DOABLE (but not preferred):
        do_something "${(@)foo}"

        # DO NOT DO THIS:
        do_something "${foo}"
        ```

In general:

* **Be kind to yourself.**

    * Write your scripts with future maintainability in mind because you will be the one coming back to your own scripts later on. Follow best practices. Use common design patterns. Choose meaningful and descriptive variable names. Document code describing what it does, not how it works. Use "TODO:" and "DEBUG:" comments freely. Be consistent. Keep things simple and straightforward.

## Closing Remarks

I did not create these rules in a vacuum. I learned by writing and maintaining many scripts, and by reviewing scripts written by others. This, however, is the first time I have written them out. I have never needed to because these all make sense to me, they feel natural, and they make my code easier to maintain. It was straightforward to look at my code and generate the style guide I have been implicitly following. It felt good to document the style guide [as a guide to my future self]({% post_url 2025-08-01--about-this-site %}).

A well documented style guide is also essential for both team environments and LLM-driven workflows. It acts as a "system prompt" for your project. By defining the standards and constraints needed to generate code that fits with the architecture, you ensure that all new code adheres to your project's design language. The net gain is that new code is aligned with your existing codebase, reducing the cost of refactoring and integration.

How would I recommend you develop your own? Review scripts you've written in the past, look at useful scripts you've run into, be critical of scripts you're working on now, and seek resources online (some large companies publish their own style guides). You will quickly learn how to organize and document your code, how to implement best practices, and what guidelines make sense for you. Don't be academic about it - be pragmatic. The goal is have a set of rules and guidelines that help you, your team, or an LLM write code that remains readable and maintainable months or years from now.