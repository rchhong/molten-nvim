# Magma

Magma is a NeoVim plugin for running code interactively with Jupyter.

<!-- TODO: add a GIF showing a demo -->

## Requirements

- NeoVim 0.5+
- Required Python packages:
    - [`pynvim`](https://github.com/neovim/pynvim) (for the Remote Plugin API)
    - [`jupyter_client`](https://github.com/jupyter/jupyter_client) (for interacting with Jupyter)
    - [`ueberzug`](https://github.com/seebye/ueberzug) (for displaying images)
    - [`Pillow`](https://github.com/python-pillow/Pillow) (also for displaying images, should be installed with `ueberzug`)

## Installation

Use your favourite package/plugin manager.

If you use `packer.nvim`,

```lua
use { 'dccsillag/magma-nvim', run = ':UpdateRemotePlugins' }
```

If you use `vim-plug`,

```vim
Plug 'dccsillag/magma-nvim', { 'do': ':UpdateRemotePlugins' }
```

Note that you will still need to configure keymappings -- see [Keybindings](#keybindings).

## Suggested settings

If you want a quickstart, these are the author's suggestions of mappings and options (beware of potential conflicts of these mappings with your own!):

```vim
nnoremap <silent> gr  :MagmaEvaluateOperator<CR>
nnoremap <silent> grr :MagmaEvaluateLine<CR>
vnoremap <silent> gr  :<C-u>MagmaEvaluateVisual<CR>
nnoremap <silent> grd :MagmaDelete<CR>
nnoremap <silent> gro :MagmaShowOutput<CR>

let g:magma_automatically_open_output = v:false
```

**Note:** Key mappings are not defined by default because of potential conflicts -- the user should decide which keys they want to use (if at all).

**Note:** The options that are altered here don't have these as their default values in order to provide a simpler (albeit perhaps a bit more inconvenient) UI for someone who just added the plugin without properly reading the README.

## Usage

The plugin provides a bunch of commands to enable interaction. It is recommended to map most of them to keys, as explained in [Keybindings](#keybindings). However, this section will refer to the commands by their names (so as to not depend on some specific mappings).

### Interface

When you execute some code, it will create a *cell*. You can recognize a cell because it will be highlighted when your cursor is in it.

A cell is delimited using two extmarks (see `:help api-extended-marks`), so it will adjust to you editing the text within it.

When your cursor is in a cell (i.e., you have an *active cell*), a floating window may be shown below the cell, reporting output. This is the *display window*, or *output window*. (To see more about whether a window is shown or not, see `:MagmaShowOutput` and `g:magma_automatically_open_output`). When you cursor is not in any cell, no cell is active.

Also, the active cell is searched for from newest to oldest. That means that you can have a cell within another cell, and if the one within is newer, then that one will be selected. (Same goes for merely overlapping cells).

The output window has a header, containing the execution count and execution state (i.e., whether the cell is waiting to be run, running, has finished successfully or has finished with an error). Below the header are shown the outputs.

Jupyter provides a rich set of outputs. To see what we can currently handle, see [Output Chunks](#output-chunks).

### Commands

#### MagmaInit

This command initializes a runtime for the current buffer. It takes a single argument, the Jupyter kernel's name. For example,

```vim
:MagmaInit python3
```

will initialize the current buffer with a `python3` kernel.

**Note:** A more friendly runtime initialization UI is planned.

#### MagmaEvaluateLine

Evaluate the current line.

Example usage:

```vim
:MagmaEvaluateLine
```

#### MagmaEvaluateVisual

Evaluate the selected text.

Example usage (after having selected some text):

```vim
:MagmaEvaluateVisual
```

#### MagmaEvaluateOperator

Evaluate the text given by some operator.

Example usage:

```vim
:MagmaEvaluateOperator
```

Upon running this, you will enter operator mode, with which you will be able to select text you want to execute. You can, of course, hit ESC to cancel, as usual with operator mode.

#### MagmaDelete

Delete the currently selected cell. (If there is no selected cell, do nothing.)

Example usage:

```vim
:MagmaDelete
```

#### MagmaShowOutput

This only makes sense when you have `g:magma_automatically_open_output = v:false`. See [Customization](#customization).

Running this command with some active cell will open the output window.

Example usage:

```vim
:MagmaShowOutput
```

## Keybindings

It is recommended to map all the evaluate commands to the same mapping (in different modes). For example, if we wanted to bind evaluation to `gr`:

```vim
nnoremap <silent> gr  :MagmaEvaluateOperator<CR>
nnoremap <silent> grr :MagmaEvaluateLine<CR>
vnoremap <silent> gr  :<C-u>MagmaEvaluateVisual<CR>
```

This way, `gr` will behave just like standard keys such as `y` and `d`.

You can, of course, also map other commands:

```vim
nnoremap <silent> grd :MagmaDelete<CR>
nnoremap <silent> gro :MagmaShowOutput<CR>
```

## Customization

Customization is done via variables.

### `g:magma_automatically_open_output`

Defaults to `v:true`.

If this is true, then whenever you have an active cell its output window will be automatically shown.

If this is false, then the output window will only be automatically shown when you've just evaluated the code. So, if you take your cursor out of the cell, and then come back, the output window won't be opened (but the cell will be highlighted). This means that there will be nothing covering your code. You can then open the output window at will using [`:MagmaShowOutput`](#magma-show-output).

## Extras

### Output Chunks

In the Jupyter protocol, most output-related messages provide a dictionary of mimetypes which can be used to display the data. Theoretically, a `text/plain` field (i.e., plain text) is always present, so we (theoretically) always have that fallback.

Here is a list of the currently handled mimetypes:

- `text/plain`: Plain text. Shown as text in the output window's buffer.
- `image/png`: A PNG image. Shown with [Ueberzug](https://github.com/seebye/ueberzug).

This already provides quite a bit of basic functionality. As development continues, more mimetypes will be added.