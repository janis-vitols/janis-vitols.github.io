---
layout: post
title:  "Replace word or selection in VIM"
date:   2016-11-16 21:45:00 +0200
categories: vim tricks
comments: true
disqus_identifier: 2A8D3BAA-5B12-4A77-8990-FCFF2175C692
---

Hello!

My favorite IDE for Ruby development is [Vim][vim]{:target="blank"} or to be more specific [NeoVim][neovim]{:target="blank"}.
Even this blog post is being written in NeoVim :smirk:

![](/images/2016/neovim-word-replacement-mappings.png)

When coding I often need to replace a word or a selection to something else globally in the same file.
I would struggle typing long commands like `:%s/\<word_to_change\>/new_word/gc` to do the job.
As you can imagine it takes some time to enter the commands...
Additionally, you have to be in the `normal mode` and do `<shift> + :` to even start typing the command.

So lets see what we can do to make word replacement easier in our editor.
Let's go deeper and investigate more about `substitute` command itself.
By checking `:help :substitute` we can [see][substitute-help]{:target="blank"}
that this command has aliases. It can be called by full name `substitute`, `su` or even shorter - just `s`.

Here is this command in full glory from help: `[range]s[ubstitute]/{pattern}/{string}/[flags] [count]`

As you can see in my example, I use `%` for `[range]`. If you don't specify any range, only current line will be searched.
As in most cases we want to change the word across the whole file, we need to specify the `%`.

Next I'm using the shortest alias `s` - you could go with `su` or `substitute` here also - but it would be even more typing :thumbsdown:.

After command itself we need to specify `{pattern}` with which we want to find word for replacement. It must be entered between slashes.
In case if no pattern is provided - `//` (left empty) the last search pattern will be used.
As you can see from my example I have provided pattern like `\<word_to_change\>` - here I'm using `\<` and `\>` around our word
to find an exact match for word `word_to_change`.

After that we are specifying `{string}` - our new word with which we want to replace. And then after last slash `/` I use `gc` `[flags]`.

`g` - *global* - Replace all occurrences in the line.  Without this argument, replacement occurs only for the first occurrence in each line.

`c` - *confirm* - Each substitution is highlighted and you must either confirm or not this substitution. In most cases I have used only 
`y` or `n`, but there are few other options also.

There are few more flags, like `i` which means *case insensitive*, but I don't use other flags very much in my day to day development.

So, as you can see, not only must you keep a lot of these things in your mind, but also there is a lot of typing and it takes time.
So I decided to create my shortcuts for this to fight this problem :muscle:.

By some Internet investigation and checking different possible ways to solve my problem, I have come up with these maps which
I have added to my `.config/nvim/init.vim` file.

{% highlight shell linenos %}
" Replace visually selected text or word (globally with confirmation)
nnoremap <C-r> :<C-U>let replacement = input('Replace word `<C-R><C-W>` with: ') <bar> %s/\<<C-R><C-W>\>/\=replacement/gc<CR>
vnoremap <C-r> y:<C-U>let replacement = input('Replace selection `<C-R>"` with: ') <bar> %s/<C-R>"/\=replacement/gc<CR>
{% endhighlight %}

You may wonder why I ended up with 2 maps here.
Well, the 1st one which uses `nnoremap` can be used when I'm in `normal mode` - replace word on which my cursor is on.
I just need to hit `CTRL + r` and I will receive prompt where I can enter new word(s) and then follow with confirmations.
The 2nd one which uses `vnoremap` can be used when I'm in `visual mode`.

Also you can see that there are some differences for these `normal mode` and `visual mode` maps.
First thing is that for `normal mode` `nnoremap` `{pattern}` uses `\<` and `\>` to find exact matching word, but for `visual mode`
`vnoremap` I don't use them, as in most cases when I do replacement in `visual mode` I'm not searching for exact match.
In `nnoremap` we use `<C-R><C-W>` to paste word which is under our cursor, but in `vnoremap` we use `y` (yank) command before prompt
for new word to copy selection in buffer and then place it in our substitute command with `<C-R>"`.
`<C-U>` is used in both of these maps to clean the commandline after `:`. `let replacement = input('Some text: ')` is just our prompt with
our text here.

And this is my solution to the problem, hope it will be usefull for someone else too. In case if you have any ideas how to do it even better,
I would be interested to hear it in comments.

[vim]:             http://www.vim.org/
[neovim]:          https://neovim.io/
[substitute-help]: http://vimdoc.sourceforge.net/htmldoc/change.html#:substitute
