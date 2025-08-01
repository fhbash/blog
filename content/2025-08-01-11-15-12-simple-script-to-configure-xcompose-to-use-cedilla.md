---
tags: post,notes,gnome,kde,fix,script
---
# Simple script to configure XCompose to use Cedilla

This is a very simple script to configure your personal ".XCompose" file and  your environment so that typing 'c will generate a cedilla c instead of an accented c.

Using Ç on Gnome/KDE sounds like issue, even if you set the right keyboard configuration. Instead to use ALT key + several keys, yo need to add this snippet into `~/.XCompose` file:

```
<dead_acute> <c> : "ç" U00E7
<dead_acute> <C> : "Ç" U00C7
```

Log out and log in again to see the results.

I made this small script file to make easy to fix, you can download using the gist link or copy/paste into a text file, make it executable by `chmod +x fix-cedilla-compose.sh` command for example.

[Complete script file on Gist](https://gist.github.com/fhbash/8f8abe491e0bcd78d6af60a48a714e47)

```bash
#!/bin/bash

# 1. Create ~/.XCompose with proper remapping
echo "Setting up ~/.XCompose for cedilla support..."

cat > "$HOME/.XCompose" <<EOF
include "%L"

<dead_acute> <c> : "ç" U00E7
<dead_acute> <C> : "Ç" U00C7
EOF

# 2. Add XCOMPOSEFILE to shell config (bash or zsh)
echo "Configuring XCOMPOSEFILE environment variable..."

SHELLRC=""
if [ -f "$HOME/.bashrc" ]; then
    SHELLRC="$HOME/.bashrc"
elif [ -f "$HOME/.zshrc" ]; then
    SHELLRC="$HOME/.zshrc"
else
    echo "No .bashrc or .zshrc found. Please export XCOMPOSEFILE manually if needed."
fi

if [ -n "$SHELLRC" ]; then
    if ! grep -q 'XCOMPOSEFILE=.*\.XCompose' "$SHELLRC"; then
        echo 'export XCOMPOSEFILE="$HOME/.XCompose"' >> "$SHELLRC"
        echo "Added XCOMPOSEFILE to $SHELLRC"
    else
        echo "XCOMPOSEFILE is already set in $SHELLRC"
    fi
fi

# 3. Reload keyboard layout
echo "Reloading keyboard layout..."
setxkbmap -option '' && setxkbmap

# 4. Apply XCOMPOSEFILE to Flatpak apps (if any)
echo "Applying XCOMPOSEFILE to Flatpak applications..."
for app in $(flatpak list --app --columns=application); do
    flatpak override --user --env=XCOMPOSEFILE="$HOME/.XCompose" "$app"
done

echo ""
echo "Compose remapping for cedilla (ç) is now configured."
echo "Please log out and log back in to ensure all changes take effect."

```

