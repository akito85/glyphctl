Error having ^M character in one of Neovim config

1. Use dos2unix
```
dos2unix /home/akito/.local/share/nvim/site/pack/packer/start/fine-cmdline.nvim/plugin/fine-cmdline.vim
```

This will pass 1 file at a time, and use 4 processors.
```
:find . -type f -print0 | xargs -0 -n 1 -P 4 dos2unix 
```

2. Use Neovim
```
%s/^M$//g

Explannation:
%s: search
/^M$: search character ^M
//g: remove character globally

or

:e ++ff=dos
:set ff=unix

or
```

3. Use sed
```
sed -i -e "s/\r//g" filename

Explanation:
-i: in-place
-e: regular expression
\r: escaped carriage return
/g: replace globally
```
