# gitlab.nvim

This Neovim plugin is designed to make it easy to review Gitlab MRs from within the editor. This means you can do things like:

- Read and Edit an MR description
- Approve or revoke approval for an MR
- Add or remove reviewers and assignees
- Resolve, reply to, and unresolve discussion threads
- Create, edit, delete, and reply to comments on an MR
- View and Manage Pipeline Jobs

And a lot more!

https://github.com/harrisoncramer/gitlab.nvim/assets/32515581/50f44eaf-5f99-4cb3-93e9-ed66ace0f675

## Requirements

- <a href="https://go.dev/">Go</a> >= v1.19

## Quick Start

1. Install Go
3. Add configuration (see Installation section)
4. Checkout your feature branch: `git checkout feature-branch`
5. Open Neovim
6. Run `:lua require("gitlab").review()` to open the reviewer pane

## Installation

With <a href="https://github.com/folke/lazy.nvim">Lazy</a>:

```lua
return {
  "harrisoncramer/gitlab.nvim",
  dependencies = {
    "MunifTanjim/nui.nvim",
    "nvim-lua/plenary.nvim",
    "sindrets/diffview.nvim",
    "stevearc/dressing.nvim", -- Recommended but not required. Better UI for pickers.
    enabled = true,
  },
  build = function () require("gitlab.server").build(true) end, -- Builds the Go binary
  config = function()
    require("gitlab").setup()
  end,
}
```

And with Packer:

```lua
use {
  'harrisoncramer/gitlab.nvim',
  requires = {
    "MunifTanjim/nui.nvim",
    "nvim-lua/plenary.nvim",
    "sindrets/diffview.nvim",
    "stevearc/dressing.nvim",
  },
  run = function() require("gitlab.server").build(true) end,
  config = function()
    require("gitlab").setup()
  end,
}
```

## Project Configuration

This plugin requires an <a href="https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html#create-a-personal-access-token">auth token</a> to connect to Gitlab. The token can be set in the root directory of the project in a `.gitlab.nvim` environment file, or can be set via a shell environment variable called `GITLAB_TOKEN` instead. If both are present, the `.gitlab.nvim` file will take precedence. 

Optionally provide a GITLAB_URL environment variable (or gitlab_url value in the `.gitlab.nvim` file) to connect to a self-hosted Gitlab instance. This is optional, use ONLY for self-hosted instances.

```
auth_token=your_gitlab_token
gitlab_url=https://my-personal-gitlab-instance.com/
```

If you don't want to write these into a dotfile, you may provide them via shell variables. These will be overridden by the dotfile if it is present:

```bash
export GITLAB_TOKEN="your_gitlab_token"
export GITLAB_URL="https://my-personal-gitlab-instance.com/"
```

## Configuring the Plugin

Here is the default setup function. All of these values are optional, and if you call this function with no values the defaults will be used:

```lua
require("gitlab").setup({
  port = nil, -- The port of the Go server, which runs in the background, if omitted or `nil` the port will be chosen automatically
  log_path = vim.fn.stdpath("cache") .. "/gitlab.nvim.log", -- Log path for the Go server
  debug = { go_request = false, go_response = false }, -- Which values to log
  attachment_dir = nil, -- The local directory for files (see the "summary" section)
  popup = { -- The popup for comment creation, editing, and replying
    exit = "<Esc>",
    perform_action = "<leader>s", -- Once in normal mode, does action (like saving comment or editing description, etc)
    perform_linewise_action = "<leader>l", -- Once in normal mode, does the linewise action (see logs for this job, etc)
},
  discussion_tree = { -- The discussion tree that holds all comments
    blacklist = {}, -- List of usernames to remove from tree (bots, CI, etc)
    jump_to_file = "o", -- Jump to comment location in file
    jump_to_reviewer = "m", -- Jump to the location in the reviewer window
    edit_comment = "e", -- Edit coment
    delete_comment = "dd", -- Delete comment
    reply = "r", -- Reply to comment
    toggle_node = "t", -- Opens or closes the discussion
    toggle_resolved = "p", -- Toggles the resolved status of the discussion
    position = "left", -- "top", "right", "bottom" or "left"
    size = "20%", -- Size of split
    relative = "editor", -- Position of tree split relative to "editor" or "window"
    resolved = '✓', -- Symbol to show next to resolved discussions
    unresolved = '✖', -- Symbol to show next to unresolved discussions
  },
  pipeline = {
    created = "",
    pending = "",
    preparing = "",
    scheduled = "",
    running = "ﰌ",
    canceled = "ﰸ",
    skipped = "ﰸ",
    success = "✓",
    failed = "",
  },
  colors = {
    discussion_tree = {
      username = 'Keyword', -- The highlight group used, for instance 'DiagnosticSignWarn'
      date = 'Comment',
      chevron = 'Comment',
    }
  }
})
```

## Usage

First, check out the branch that you want to review locally.

```
git checkout feature-branch
```

Then open Neovim. To begin, try running the `summary` command or the `review` command.

### Summary

The `summary` action will open the MR title and description. 

```lua
require("gitlab").summary()
```

After editing the description or title, you may save your changes via the `settings.popup.perform_action` keybinding.

### Reviewing Diffs

The `review` action will open a diff of the changes. You can leave comments using the `create_comment` action. In visual mode, add multiline comments with the `create_multiline_comment` command, and add suggested changes with the `create_comment_suggestion` command.

```lua
require("gitlab").review()
require("gitlab").create_comment()
require("gitlab").create_multiline_comment()
require("gitlab").create_comment_suggestion()
```

For suggesting changes you can use `create_comment_suggestion` in visual mode which works similar to `create_multiline_comment` but prefills the comment window with Gitlab's [suggest changes](https://docs.gitlab.com/ee/user/project/merge_requests/reviews/suggestions.html) code block with prefilled code from the visual selection.

### Discussions and Notes

Gitlab groups threads of comments together into "discussions."

To display all discussions for the current MR, use the `toggle_discussions` action, which will show the discussions in a split window.

```lua
require("gitlab").toggle_discussions()
```

You can jump to the comment's location in the reviewer window by using the `state.settings.discussion_tree.jump_to_reviewer` key, or the actual file with the 'state.settings.discussion_tree.jump_to_file' key.

Within the discussion tree, you can delete/edit/reply to comments with the `state.settings.discussion_tree.SOME_ACTION` keybindings.

#### Notes

If you'd like to create a note in an MR (like a comment, but not linked to a specific line) use the `create_note` action. The same keybindings for delete/edit/reply are available on the note tree.

```lua
require("gitlab").create_note()
```

### Uploading Files

To attach a file to an MR description, reply, comment, and so forth use the `settings.popup.perform_linewise_action` keybinding when the the popup is open. This will open a picker that will look in the directory you specify in the `settings.attachment_dir` folder (this must be an absolute path) for files.

When you have picked the file, it will be added to the current buffer at the current line.

Use the `settings.popup.perform_action` to send the changes to Gitlab.

### MR Approvals

You can approve or revoke approval for an MR with the `approve` and `revoke` actions respectively.

```lua
require("gitlab").approve()
require("gitlab").revoke()
```

### Pipelines

You can view the status of the pipeline for the current MR with the `pipeline` action.

```lua
require("gitlab").pipeline()
```

To re-trigger failed jobs in the pipeline manually, use the `settings.popup.perform_action` keybinding. To open the log trace of a job in a new Neovim buffer, use your `settings.popup.perform_linewise_action` keybinding.

### Reviewers and Assignees

The `add_reviewer` and `delete_reviewer` actions, as well as the `add_assignee` and `delete_assignee` functions, will let you choose from a list of users who are availble in the current project:

```lua
require("gitlab").add_reviewer()
require("gitlab").delete_reviewer()
require("gitlab").add_assignee()
require("gitlab").delete_assignee()
```

These actions use Neovim's built in picker, which is much nicer if you install <a href="https://github.com/stevearc/dressing.nvim">dressing</a>. If you use Dressing, please enable it:

```lua
require("dressing").setup({
    input = {
        enabled = true
    }
})
```

## Keybindings

The plugin does not set up any keybindings outside of these buffers, you need to set them up yourself. Here's what I'm using:

```lua
local gitlab = require("gitlab")
vim.keymap.set("n", "<leader>glr", gitlab.review)
vim.keymap.set("n", "<leader>gls", gitlab.summary)
vim.keymap.set("n", "<leader>glA", gitlab.approve)
vim.keymap.set("n", "<leader>glR", gitlab.revoke)
vim.keymap.set("n", "<leader>glc", gitlab.create_comment)
vim.keymap.set("v", "<leader>glc", gitlab.create_multiline_comment)
vim.keymap.set("v", "<leader>glC", gitlab.create_comment_suggestion)
vim.keymap.set("n", "<leader>gln", gitlab.create_note)
vim.keymap.set("n", "<leader>gld", gitlab.toggle_discussions)
vim.keymap.set("n", "<leader>glaa", gitlab.add_assignee)
vim.keymap.set("n", "<leader>glad", gitlab.delete_assignee)
vim.keymap.set("n", "<leader>glra", gitlab.add_reviewer)
vim.keymap.set("n", "<leader>glrd", gitlab.delete_reviewer)
vim.keymap.set("n", "<leader>glp", gitlab.pipeline)
vim.keymap.set("n", "<leader>glo", gitlab.open_in_browser)
```

## Troubleshooting

**To check that the current settings of the plugin are configured correctly, please run: `:lua require("gitlab").print_settings()`**

This plugin uses a Golang server to reach out to Gitlab. It's possible that something is going wrong when starting that server or connecting with Gitlab. The Golang server runs outside of Neovim, and can be interacted with directly in order to troubleshoot. To start the server, check out your feature branch and run these commands:

```lua
:lua require("gitlab.server").build(true)
:lua require("gitlab.server").start(function() print("Server started") end)
```

The easiest way to debug what's going wrong is to turn on the `debug` options in your setup function. This will allow you to see requests leaving the Go server, and the responses coming back from Gitlab. Once the server is running, you can also interact with the Go server like any other process:

```
curl --header "PRIVATE-TOKEN: ${GITLAB_TOKEN}" localhost:21036/info
```

## Extra Goodies

If you are like me and want to quickly switch between recent branches and recent merge request reviews and assignments, check out the git scripts contained <a href="https://github.com/harrisoncramer/.dotfiles/blob/main/scripts/bin/git-reviews">here</a> and <a href="https://github.com/harrisoncramer/.dotfiles/blob/main/scripts/bin/git-authored">here</a> for inspiration.
