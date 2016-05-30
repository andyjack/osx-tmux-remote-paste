## Local (OS X) Side

#### `~/Library/LaunchAgents/pbcopy.plist`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
     <key>Label</key>
     <string>localhost.pbcopy</string>
     <key>ProgramArguments</key>
     <array>
         <string>/usr/bin/pbcopy</string>
     </array>
     <key>inetdCompatibility</key>
     <dict>
          <key>Wait</key>
          <false/>
     </dict>
     <key>Sockets</key>
     <dict>
          <key>Listeners</key>
               <dict>
                    <key>SockServiceName</key>
                    <string>2224</string>
                    <key>SockNodeName</key>
                    <string>127.0.0.1</string>
               </dict>
     </dict>
</dict>
</plist>
```

#### `~/Library/LaunchAgents/pbpaste.plist`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
     <key>Label</key>
     <string>localhost.pbpaste</string>
     <key>ProgramArguments</key>
     <array>
         <string>/usr/bin/pbpaste</string>
     </array>
     <key>inetdCompatibility</key>
     <dict>
          <key>Wait</key>
          <false/>
     </dict>
     <key>Sockets</key>
     <dict>
          <key>Listeners</key>
               <dict>
                    <key>SockServiceName</key>
                    <string>2225</string>
                    <key>SockNodeName</key>
                    <string>127.0.0.1</string>
               </dict>
     </dict>
</dict>
</plist>
```

#### `~/.ssh/config`
```
Host myhost
    HostName 192.168.1.123
    User myname
    RemoteForward 2224 127.0.0.1:2224
    RemoteForward 2225 127.0.0.1:2225
```

**After adding the PLists above, you'll have to run:**

```bash
launchctl load ~/Library/LaunchAgents/pbcopy.plist
launchctl load ~/Library/LaunchAgents/pbpaste.plist
```

## Remote (Linux) Side

#### `~/.tmux.conf`
```
if-shell 'test "$(uname)" = "Linux"' 'source ~/.tmux-linux.conf'
```

#### `~/.tmux-linux.conf`
```
bind C-c run "tmux save-buffer - | pbcopy-remote"
bind C-v run "tmux set-buffer $(pbpaste-remote); tmux paste-buffer"
```

#### `~/bin/pbpaste-remote`
```bash
#!/bin/sh
nc localhost 2225
```

#### `~/bin/pbcopy-remote`
```bash
#!/bin/sh
cat | nc -q1 localhost 2224
```

#### `~/.vimrc`
```viml
function! PropagatePasteBufferToOSX()
  let @n=getreg("*")
  call system('pbcopy-remote', @n)
  echo "done"
endfunction

function! PopulatePasteBufferFromOSX()
  let @+ = system('pbpaste-remote')
  echo "done"
endfunction

nnoremap <leader>6 :call PopulatePasteBufferFromOSX()<cr>
nnoremap <leader>7 :call PropagatePasteBufferToOSX()<cr>
```