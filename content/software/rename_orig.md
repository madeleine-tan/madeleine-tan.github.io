+++
title = "File rename original"
hideAuthor = true
hideDate = true
hideLastMod = true
+++

```bash
#!/bin/bash

# ------------------------------
# Rename image files based on user inputs and image timestamp
# MT April 2023
# ------------------------------

FolderNew="for_upload"					# renamed files will be saved here
FolderOrig="original"					# original files will be copied to here
mkdir -p "$FolderNew"
mkdir -p "$FolderNew"/"$FolderOrig"
cp *.HEIC "$FolderNew"
cp *.HEIC "$FolderNew"/"$FolderOrig"
cd $FolderNew							# work in renamed files directory

counter=0

total=$(find ./ -maxdepth 1 -type f -name "*.HEIC" | wc -l)

read -p 'Input site location: ' siteloc												# user input for location, populates first part of file name
read -p 'Input additional info (install, extract, etc.): ' action					# user input for action, populates last part of file name

printf "Renaming $total files for $siteloc, $action ..."
printf "\n"

for i in *.HEIC		# main code
	do
		creation=$(sips -g creation "$i")
		time=$(echo "${creation: -8:8}" | sed s/:/_/g)
		date=$(echo "${creation: -19:10}" | sed s/:/_/g)
		printf "\n"
		filename="$siteloc"_"$date"_"$time"_"$action".HEIC
		counter=$((counter+1))
		mv $i $filename
		printf "$i --> $filename"
		printf "\n"
	done

cd ..
rm *.HEIC		# this will nuke files outside of $FolderNew and $FolderOrig
printf "\n"
printf "Renaming complete for $siteloc, $action."
printf "\n"
```
