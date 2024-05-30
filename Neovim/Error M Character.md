Error having ^M character in one of Neovim config

1. Use dos2unix
```
dos2unix /home/akito/.local/share/nvim/site/pack/packer/start/fine-cmdline.nvim/plugin/fine-cmdline.vim
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
