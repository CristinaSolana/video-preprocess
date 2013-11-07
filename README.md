1. Install Handbrake CLI: http://handbrake.fr/downloads2.php (Must place in Applications directory)
2. Open Automator, Create New Folder Action
3. Choose folder for unprocessed previews
4. Add AppleScript, paste:
```
on run {input, parameters}
	set handbrakeCli to "/Applications/HandBrakeCLI"
	set defaultPreset to 7
	
	-- Make sure HandBrakeCLI is installed
	set handbrakeInstalled to false
	tell application "Finder" to if exists handbrakeCli as POSIX file then set handbrakeInstalled to true
	if not handbrakeInstalled then
		display alert "You need HandBrake CLI" message "Download, install and move HandBrakeCLI into your Applications. http://handbrake.fr/downloads2.php" as critical
		return
	end if
	
	--set the destination to POSIX path of (choose folder with prompt "Select the conversion destination")
	set destination to POSIX path of ((system attribute "HOME") & "/Desktop/previews-processed/")
	
	repeat with inputFile in input
		
		-- TODO: Test file width before converting, if smaller than 720 just move/copy
		
		-- Get original file name
		set thePath to POSIX path of inputFile
		set AppleScript's text item delimiters to "/"
		
		-- Get file name
		set inputFileName to (item -1 of (every text item of thePath)) as text
		
		-- Get file name without extension
		set AppleScript's text item delimiters to {"-", ".", "[", "]", "{", "}", ":", ";", ",", "/", "|", "!", "@", "#", "$", "%", "^", "&", "*", "(", ")", "+"}
		set inputFileNoExtension to items 1 through -2 of (every text item of inputFileName)
		
		-- Replace special characters with underscore
		set outputFileName to ""
		repeat with fileNamePart in inputFileNoExtension
			set outputFileName to outputFileName & fileNamePart & "_"
			set outputFileNameSubString to text 1 thru -2 of outputFileName
		end repeat
		
		set handbrakeCommand to "nice " & handbrakeCli & " -i " & quoted form of (POSIX path of inputFile) & " -o " & quoted form of (destination & outputFileNameSubString & ".mp4") & " -e x264  -q 20.0 -r 30 --pfr  -a 1 -E faac -B 160 -6 dpl2 -R Auto -D 0.0 --audio-copy-mask aac,ac3,dtshd,dts,mp3 --audio-fallback ffac3 -f mp4 -4 -maxwidth 720 --loose-anamorphic --modulus 2 -m --x264-preset medium --h264-profile high --h264-level 3.1 --optimize"
		--return handbrakeCommand
		do shell script handbrakeCommand
	end repeat
	return input
end run
```
5. Press run
6. Create directory named previews-processed in /Users/[your-user]/Desktop
