# Extended session management for Vim

The `session.vim` plug-in improves upon [Vim](http://www.vim.org/)'s built-in [:mksession][mksession] command by enabling you to easily and (if you want) automatically persist and restore your Vim editing sessions. It works by generating a [Vim script](http://vimdoc.sourceforge.net/htmldoc/usr_41.html#script) that restores your current settings and the arrangement of tab pages and/or split windows and the files they contain.

To persist your current editing session you can execute the `:SaveSession` command. If you don't provide a name for the session 'default' is used (you can change this name with an option). You're free to use whatever characters you like in session names. When you want to restore your session simply execute `:OpenSession`. Again the name 'default' is used if you don't provide one. When a session is active, has been changed and you quit Vim you'll be prompted whether you want to save the open session before quitting Vim:

![Screenshot of auto-save prompt](http://peterodding.com/code/vim/session/autosave.png)

When you start Vim without editing any files and the default session exists, you'll be prompted whether you want to restore the default session:

![Screenshot of auto-open prompt](http://peterodding.com/code/vim/session/autoopen.png)

When you start Vim with a custom [server name](http://vimdoc.sourceforge.net/htmldoc/remote.html#--servername) that matches one of the existing session names then the matching session will be automatically restored. For example I use several sessions to quickly edit my Vim plug-ins:

    $ gvim --servername easytags-plugin
    $ gvim --servername session-plugin
    $ # etc.

The session scripts created by this plug-in are stored in the directory `~/.vim/sessions` (on UNIX) or `~\vimfiles\sessions` (on Windows) but you can change the location by setting `g:session_directory`. If you're curious what the session scripts generated by `session.vim` look like see the [sample below](http://peterodding.com/code/vim/session/#sample_session_script).

## Installation

Unzip the most recent [ZIP archive](http://peterodding.com/code/vim/downloads/session.zip) file inside your Vim profile directory (usually this is `~/.vim` on UNIX and `%USERPROFILE%\vimfiles` on Windows), restart Vim and execute the command `:helptags ~/.vim/doc` (use `:helptags ~\vimfiles\doc` instead on Windows). After you restart Vim the following commands will be available to you:

## Commands

Note that environment variables inside command arguments are expanded by the plug-in.

### The `:SaveSession` command

This command saves your current editing session just like Vim's built-in [:mksession][mksession] command does. The difference is that you don't pass a full pathname as argument but just a name, any name really. Press `<Tab>` to get completion of existing session names. If you don't provide an argument the default session name is used, unless an existing session is open in which case the name of that session will be used.

If the session you're trying to save is already active in another Vim instance you'll get a warning and nothing happens. You can use a bang (!) as in `:SaveSession! ...` to ignore the warning and save the session anyway.

Your session script will be saved in the directory pointed to by the `g:session_directory` option.

### The `:OpenSession` command

This command is basically [:source][source] in disguise, but it supports tab completion of session names and it executes `:CloseSession` before opening the session. When you don't provide a session name and only a single session exists then that session is opened, otherwise the plug-in will ask you to select one from a list:

    Please select the session to restore:
    
     1. vim-profile
     2. session-plugin
     3. etc.
    
    Type number and <Enter> or click with mouse (empty cancels):

If the session you're trying to open is already active in another Vim instance you'll get a warning and nothing happens. You can use a bang (!) as in `:OpenSession! ...` to ignore the warning and open the session anyway.

Note also that when you use a bang (!) right after the command name existing tab pages and windows are closed, discarding any changes in the files you were editing!

### The `:RestartVim` command

This command saves your current editing session, restarts Vim and restores your editing session. This can come in handy when you're debugging Vim scripts which can't be easily/safely [reloaded using a more lightweight approach](http://peterodding.com/code/vim/reload/). It should work fine on Windows and UNIX alike but because of technical limitations it only works in graphical Vim.

Any commands following the `:RestartVim` command are intercepted and executed after Vim is restarted and your session has been restored. This makes it easy to perform manual tests which involve restarting Vim, e.g. `:RestartVim | edit /path/to/file | call MyTest()`.

### The `:CloseSession` command

This command closes all but the current tab page and window and then edits a new, empty buffer. If a session is loaded when you execute this command the plug-in will first ask you whether you want to save that session.

Note that when you use a bang (!) right after the command name existing tab pages and windows are closed, discarding any changes in the files you were editing!

### The `:DeleteSession` command

Using this command you can delete any of the sessions created by this plug-in. If the session you are trying to delete is currently active in another Vim instance you'll get a warning and nothing happens. You can use a bang (!) as in `:DeleteSession! ...` to ignore the warning and delete the session anyway.

Note that this command only deletes the session script, it leaves your open tab pages and windows exactly as they were.

### The `:ViewSession` command

Execute this command to view the Vim script generated for a session. This command is useful when you need to review the generated Vim script repeatedly, for example while debugging or modifying the `session.vim` plug-in.

## Options

### The `sessionoptions` setting

Because the `session.vim` plug-in uses Vim's [:mksession][mksession] command you can change how it works by setting ['sessionoptions'][sessionoptions] in your [vimrc script] [vimrc], for example:

    " If you only want to save the current tab page:
    set sessionoptions-=tabpages

    " If you don't want help windows to be restored:
    set sessionoptions-=help

Note that the session.vim plug-in automatically and unconditionally executes the following change just before saving a session:

    " Don't persist options and mappings because it can corrupt sessions.
    set sessionoptions-=options

### The `g:session_directory` option

This option controls the location of your session scripts. Its default value is `~/.vim/sessions` (on UNIX) or `~\vimfiles\sessions` (on Windows). If you don't mind the default you don't have to do anything; the directory will be created for you. Note that a leading `~` is expanded to your current home directory (`$HOME` on UNIX, `%USERPROFILE%` on Windows).

### The `g:session_default_name` option

The name of the default session without directory or filename extension (you'll never guess what the default is).

### The `g:session_extension` option

The filename extension of session scripts. This should include the dot that separates the basename from the extension. Defaults to '.vim'.

### The `g:session_autoload` option

By default this option is set to `'prompt'`. This means that when you start Vim without opening any files and the default session script exists, the session plug-in will ask whether you want to restore your default session. When you set this option to `'yes'` and you start Vim without opening any files the default session will be restored without a prompt. To completely disable automatic loading you can set this option to `'no'`.

### The `g:session_autosave` option

By default this option is set to `'prompt'`. When you've opened a session and you quit Vim, the session plug-in will ask whether you want to save the changes to your session. Set this option to `'yes'` to always automatically save open sessions when you quit Vim. To completely disable automatic saving you can set this option to `'no'`.

### The `g:session_verbose_messages` option

The session load/save prompts are quite verbose by default because they explain how to disable the prompts. If you find the additional explanation distracting you can lower the verbosity by setting this option to 0 (false) in your [vimrc script] [vimrc].

### The `g:session_default_to_last` option

By default this option is set to false (0). When you set this option to true (1) and you start Vim, the session plug-in will open your last used session instead of the default session. Note that the session plug-in will still show you the dialog asking whether you want to restore the last used session. To get rid of the dialog you have to set `g:session_autoload` to `'yes'`.

### The `g:session_persist_globals` option

The session.vim plug-in uses Vim's [:mksession] [mksession] command but it changes ['sessionoptions'][sessionoptions] so that Vim options and mappings are not persisted. The plug-in does this because persistence of options and mappings can break loading of sessions, in other words it's fragile (in my opinion).

If you want the plug-in to persist specific global variables or options you can add their names to the list `g:session_persist_globals` in your [vimrc script] [vimrc]:

    " Persist the value of the global option 'makeprg'.
    let g:session_persist_globals = ['&makeprg']

Because the [vimrc script] [vimrc] is loaded before the plug-in you have to define the list yourself. To persist multiple values:

    " Persist all options related to :make.
    let g:session_persist_globals = ['&makeprg', '&makeef']

Here's how you persist global variables: (in this case the variables of the session plug-in itself :-)

    " Persist the options of the session plug-in using the session plug-in...
    let g:session_persist_globals = ['&sessionoptions']
    call add(g:session_persist_globals, 'g:session_autoload')
    call add(g:session_persist_globals, 'g:session_autosave')
    call add(g:session_persist_globals, 'g:session_default_to_last')
    call add(g:session_persist_globals, 'g:session_persist_globals')

The example above doesn't persist the `g:session_directory` variable because this variable is used before loading a session script so persisting it inside the session script is pointless.

### The `g:session_restart_environment` option

This option is a list of environment variable names (without the dollar signs) that the `:RestartVim` command will pass on to the new instance of Vim. This option is only useful on UNIX. By default the three environment variables `$TERM`, `$VIM` and `$VIMRUNTIME` are included in this list.

### The `g:session_command_aliases` option

The names of the commands defined by the session plug-in start with the action they perform, followed by the string 'Session'. Some people prefer it the other way around because they find it easier to remember and you can type `:Session<Tab>` to get completion of all available commands (actually this works with the other style as well if you type `:*Session<Tab>` but I digress). If you are one of those people you can enable this option in your [vimrc script] [vimrc] like this:

    :let g:session_command_aliases = 1

When this option is enabled the session plug-in will define the following command aliases:

 * `SessionOpen` is an alias for `OpenSession`
 * `SessionView` is an alias for `ViewSession`
 * `SessionSave` is an alias for `SaveSession`
 * `SessionDelete` is an alias for `DeleteSession`
 * `SessionClose` is an alias for `CloseSession`

The aliases support tab completion just like the real commands; they're exactly the same except for the names.

When you enable the aliases, the default command names will still be available. If you really don't like them, feel free to delete them using [:delcommand] [delcommand].

### The `g:loaded_session` option

This variable isn't really an option but if you want to avoid loading the `session.vim` plug-in you can set this variable to any value in your [vimrc script] [vimrc]:

    :let g:loaded_session = 1

## Compatibility with other plug-ins

Vim's [:mksession][mksession] command isn't really compatible with plug-ins that create buffers with generated content and because of this `session.vim` includes specific workarounds for such plug-ins:

 * [BufExplorer](http://www.vim.org/scripts/script.php?script_id=42), [Project](http://www.vim.org/scripts/script.php?script_id=69) and [NERD tree](http://www.vim.org/scripts/script.php?script_id=1658) windows are supported;
 * When [shell.vim](http://peterodding.com/code/vim/shell/) is installed Vim's full-screen state is persisted;
 * The [netrw](http://vimdoc.sourceforge.net/htmldoc/pi_netrw.html#netrw-start) and [taglist.vim](http://www.vim.org/scripts/script.php?script_id=273) plug-ins support sessions out of the box.

If your favorite plug-in doesn't work with `session.vim` drop me a mail and I'll see what I can do. Please include a link to the plug-in in your e-mail so that I can install and test the plug-in.

## Known issues

Recently this plug-in switched from reimplementing [:mksession][mksession] to actually using it because this was the only way to support complex split window layouts. Only after making this change did I realize [:mksession][mksession] doesn't support [quickfix](http://vimdoc.sourceforge.net/htmldoc/quickfix.html#quickfix) and [location list](http://vimdoc.sourceforge.net/htmldoc/quickfix.html#location-list) windows and of course it turns out that bolting on support for these after the fact is going to complicate the plug-in significantly (in other words, I'm working on it but it might take a while...)

## Contact

If you have questions, bug reports, suggestions, etc. the author can be contacted at <peter@peterodding.com>. The latest version is available at <http://peterodding.com/code/vim/session/> and <http://github.com/xolox/vim-session>. If you like the script please vote for it on [Vim Online](http://www.vim.org/scripts/script.php?script_id=3150).

## License

This software is licensed under the [MIT license](http://en.wikipedia.org/wiki/MIT_License).  
© 2013 Peter Odding &lt;<peter@peterodding.com>&gt;.

## Sample session script

Here's an example session script generated by the `session.vim` plug-in while I was editing the plug-in itself in Vim:

    " ~/.vim/sessions/example.vim: Vim session script.
    " Created by session.vim on 30 August 2010 at 05:26:28.
    " Open this file in Vim and run :source % to restore your session.
    
    set guioptions=aegit
    set guifont=Monaco\ 13
    if exists('g:syntax_on') != 1 | syntax on | endif
    if exists('g:did_load_filetypes') != 1 | filetype on | endif
    if exists('g:did_load_ftplugin') != 1 | filetype plugin on | endif
    if exists('g:did_indent_on') != 1 | filetype indent on | endif
    if !exists('g:colors_name') || g:colors_name != 'slate' | colorscheme slate | endif
    call setqflist([])
    let SessionLoad = 1
    if &cp | set nocp | endif
    let s:so_save = &so | let s:siso_save = &siso | set so=0 siso=0
    let v:this_session=expand("<sfile>:p")
    silent only
    cd ~/Development/Vim/vim-session
    if expand('%') == '' && !&modified && line('$') <= 1 && getline(1) == ''
      let s:wipebuf = bufnr('%')
    endif
    set shortmess=aoO
    badd +473 ~/Development/Vim/vim-session/autoload.vim
    badd +1 ~/Development/Vim/vim-session/README.md
    badd +1 ~/Development/Vim/vim-session/session.vim
    badd +1 ~/Development/Vim/vim-session/TODO.md
    set lines=43 columns=167
    edit ~/Development/Vim/vim-session/README.md
    set splitbelow splitright
    set nosplitbelow
    set nosplitright
    wincmd t
    set winheight=1 winwidth=1
    argglobal
    let s:l = 28 - ((27 * winheight(0) + 21) / 42)
    if s:l < 1 | let s:l = 1 | endif
    exe s:l
    normal! zt
    28
    normal! 0
    tabedit ~/Development/Vim/vim-session/TODO.md
    set splitbelow splitright
    set nosplitbelow
    set nosplitright
    wincmd t
    set winheight=1 winwidth=1
    argglobal
    let s:l = 6 - ((5 * winheight(0) + 21) / 42)
    if s:l < 1 | let s:l = 1 | endif
    exe s:l
    normal! zt
    6
    normal! 0
    tabedit ~/Development/Vim/vim-session/session.vim
    set splitbelow splitright
    set nosplitbelow
    set nosplitright
    wincmd t
    set winheight=1 winwidth=1
    argglobal
    let s:l = 17 - ((16 * winheight(0) + 21) / 42)
    if s:l < 1 | let s:l = 1 | endif
    exe s:l
    normal! zt
    17
    normal! 014l
    tabedit ~/Development/Vim/vim-session/autoload.vim
    set splitbelow splitright
    set nosplitbelow
    set nosplitright
    wincmd t
    set winheight=1 winwidth=1
    argglobal
    let s:l = 473 - ((41 * winheight(0) + 21) / 42)
    if s:l < 1 | let s:l = 1 | endif
    exe s:l
    normal! zt
    473
    normal! 018l
    tabnext 4
    if exists('s:wipebuf')
      silent exe 'bwipe ' . s:wipebuf
    endif
    unlet! s:wipebuf
    set winheight=1 winwidth=1 shortmess=filnxtToO
    let s:sx = expand("<sfile>:p:r")."x.vim"
    if file_readable(s:sx)
      exe "source " . fnameescape(s:sx)
    endif
    let &so = s:so_save | let &siso = s:siso_save
    doautoall SessionLoadPost
    unlet SessionLoad


[delcommand]: http://vimdoc.sourceforge.net/htmldoc/map.html#:delcommand
[mksession]: http://vimdoc.sourceforge.net/htmldoc/starting.html#:mksession
[sessionoptions]: http://vimdoc.sourceforge.net/htmldoc/options.html#%27sessionoptions%27
[source]: http://vimdoc.sourceforge.net/htmldoc/repeat.html#:source
[vimrc]: http://vimdoc.sourceforge.net/htmldoc/starting.html#vimrc
