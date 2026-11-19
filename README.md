# MacBook battery calibration workaround for macOS 27+

My MacBook kept entering battery calibration mode on macOS 27. Two Terminal commands and one restart got it out of that state. If the same behavior returns, repeat both commands and restart once after the second command succeeds. Compatibility with later macOS releases is unverified.

## 1. Check the battery-gauging logs

Open Terminal and run this command while the issue is happening.

```sh
/usr/bin/log show --last 10m --style compact --predicate 'process == "PowerUIAgent"' | /usr/bin/grep -Ei 'ChargingUpForGauging|mitigations are in place'
```

This reads the last 10 minutes of `PowerUIAgent` logs and prints lines containing either `ChargingUpForGauging` or `mitigations are in place`, ignoring capitalization. It only reads logs. If it finishes without an error and prints nothing, neither phrase appeared in the entries it searched. A match shows what the process logged during that window.

## 2. Back up and move the battery-gauging preference file

Paste the entire block into Terminal. Enter your Mac administrator password if prompted.

```sh
plist="/private/var/root/Library/Preferences/com.apple.powerd.batterygauging.plist"
stamp=$(/bin/date +%Y%m%d-%H%M%S)
backup="$HOME/Desktop/batterygauging-$stamp.plist.bak"
moved="$HOME/Desktop/batterygauging-$stamp.plist.moved"

if sudo /usr/bin/stat -f 'Found: %N' "$plist"; then
  sudo /bin/cp -p "$plist" "$backup" &&
    sudo /bin/mv "$plist" "$moved" &&
    echo "Backup: $backup" &&
    echo "Original moved to: $moved"
else
  echo "Could not find or read the file; nothing changed."
fi
```

The block checks for the preference file and copies it to your Desktop as a timestamped `.plist.bak` backup. Only after the copy succeeds does it move the original out of the system preferences folder and onto your Desktop as `.plist.moved`. It prints both paths when those steps succeed.

## 3. Restart once

When Terminal prints both `Backup:` and `Original moved to:`, choose **Apple menu  → Restart**. After signing back in, check whether your MacBook has left calibration mode. If the block reports an error or a missing file, stop before restarting and review the message.
