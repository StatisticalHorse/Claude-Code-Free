Help me set up local PC access for Claude Desktop on Windows, so Claude can work on files and run PowerShell commands, but only inside one folder I choose. I'm not technical and don't understand JSON, so go one step at a time in plain language and wait for me to confirm each step. Don't dump everything at once. You can't run anything on my PC from this chat, so give me exact steps to follow.

BEFORE STARTING, check: I'm on Windows, I have Claude Desktop, and Node.js is installed (have me run `node -v` in PowerShell; if missing, send me to nodejs.org to install the LTS version).

STEP 1 - Scripts folder on PATH
Have me create a folder such as C:\Users\<my username>\bin and add it to my user PATH (Start menu > "Edit environment variables for your account" > Path > New). Then have me close and reopen PowerShell.

STEP 2 - Create two files in that folder (Notepad, Save as type "All files", exact names)

File 1: claude-here.cmd
~~~
@echo off
rem Sets the current folder as the only folder Claude may work in.
if not exist "%USERPROFILE%\.claude-here" mkdir "%USERPROFILE%\.claude-here"
>"%USERPROFILE%\.claude-here\root.txt" echo %CD%
echo Claude root set to: %CD%
echo Fully quit and reopen Claude Desktop to apply.
~~~

File 2: claude-here-server.cmd
~~~
@echo off
rem Started by Claude Desktop. Usage: claude-here-server.cmd fs|shell
setlocal
set "ROOT_FILE=%USERPROFILE%\.claude-here\root.txt"
if not exist "%ROOT_FILE%" (
  echo No root folder set. Run claude-here in a folder first. 1>&2
  exit /b 1
)
set /p ROOT=<"%ROOT_FILE%"
if /i "%~1"=="fs" (
  npx -y @modelcontextprotocol/server-filesystem "%ROOT%"
  exit /b %errorlevel%
)
if /i "%~1"=="shell" (
  npx -y wcli0 --shell powershell --allowedDir "%ROOT%" --initialDir "%ROOT%"
  exit /b %errorlevel%
)
echo Unknown mode "%~1". Use fs or shell. 1>&2
exit /b 1
~~~

STEP 3 - Claude Desktop config
Have me open Claude Desktop > Settings > Developer > Edit Config (this creates claude_desktop_config.json if missing). Do NOT have me overwrite the file. Ask me to paste its current contents here, then give me back the complete merged file to copy over everything, with this block added inside the top-level braces, after a comma:

"mcpServers": {
  "pc-files": { "command": "cmd", "args": ["/c", "claude-here-server.cmd", "fs"] },
  "pc-shell": { "command": "cmd", "args": ["/c", "claude-here-server.cmd", "shell"] }
}

If a "mcpServers" section already exists, add just the two entries to it.

STEP 4 - Choose the folder and restart
Have me run in PowerShell: cd <the folder I want Claude to work in>, then claude-here. Then fully quit and reopen Claude Desktop (tray icon > Quit). The folder is read at app start, so repeat this whenever I want a different folder.

STEP 5 - Verify (in a NEW chat inside Claude Desktop)
Ask Claude there to list the allowed directories and the files in the folder. Then test the limit: ask it to list a folder outside the root; the file tool should refuse.

TROUBLESHOOTING
- "Disconnected" errors: have me run `claude-here-server.cmd fs` in PowerShell. "Not recognized" means PATH problem (reopen PowerShell). "No root folder set" means I haven't run claude-here yet. Logs are in %APPDATA%\Claude\logs (mcp-server-*.log). Package names and flags can change, so check current docs if npx errors appear.
- Config file location can differ by install; always use Settings > Developer > Edit Config.

RULES FOR CLAUDE ONCE SET UP
- Work only inside the approved folder. Ask me before deleting anything, installing anything, or editing files outside it.
- The shell server only limits the starting folder, not paths written inside commands, so it is not a true sandbox. Never use paths outside the root in commands, and prefer the file tools for file work.
- Keep replies short.
