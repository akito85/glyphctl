This is my productivity configuration
<br/>

FISH
===

**Install Fish**
```
I use deb package from ubuntu port (not via apt, **outdated**)
```

**Install Fish base utility**
```
1. Fisher
2. Exa
3. Fzf
4. Nvm
```
___

RUST
===

**Install Rust**
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

**Config Rust to Fish Shell environment**
```
set -Ua fish_user_paths $HOME/.cargo/bin
```

**Install Rust base utility**
```
1. Cargo-update 
2. Alacritty
3. Starship
4. Neovide
5. Exa
6. Bat
7. Fd-find
8. Zoxide
9. Ripgrep
10. Broot
11. Xh
12. Git-delta
13. Grex
14. Zellij
15. Procs
```
___

PYTHON
===

**Install Fish**
```
sudo apt install python3 python3-pip
```

**Install Python base utility**
```
1. Bpytop
2. Ranger-fm
3. Httpie
```

NEOVIM / NEOVIDE
===

**Neovim Shortcuts**
```
PREFIX: CTRL + SPACE
MOVEMENTS:
	H: LEFT
	J: DOWN
	K: TOP
	L: RIGHT

FIND AND REPLACE ONE LINE:
:s/<find_string>/<replace_string>/g

FIND AND REPLACE ALL LINES:
:%s/<find_string>/<replace_string>/g
```

**Neovim Plugins Shortcuts**
- Neotree
```
PREFIX: SPACE
ADD DIRECTORY: PREFIX + SHIFT + A
PREVIEW FILE: PREFIX + SHIFT + P
OPEN FILE HSPLIT: PREFIX + SHIFT + S
OPEN FILE VSPLIT: PREFIX + S
OPEN FILE TAB NEW: PREFIX + T
OPEN NODE/FILE/FOLDER: ENTER
CLOSE NODE: PREFIX + SHIFT + C
CLOSE ALL NODE: PREFIX + Z
ADD FILE: PREFIX + A
ADD DIRECTORY: PREFIX + SHIFT + A
DELETE FILE/DIRECTORY: PREFIX + D
RENAME FILE/DIRECTORY: PREFIX + R
COPY TO CLIPBOARD FILE/DIRECTORY: PREFIX + Y
CUT TO CLIPBOARD FILE/DIRECTORY: PREFIX + X
PASTE FROM CLIPBOARD: PREFIX + P
COPY FILE/DIRECTORY: PREFIX + C
MOVE FILE/DIRECTORY: PREFIX + M
CLOSE WINDOW: PREFIX + Q
REFRESH WINDOW: PREFIX + R
SHOW FILE DETAILS: PRFIX + I
PREV SOURCE/BUFFER: PREFIX + SHIFT + <
NEXT SOURCE/BUFFER: PREFIX + SHIFT + >
```