
<h2>Description</h2>

"In this lab, I'll walk you through the process of setting up a file integrity monitor using live coding. By demonstrating the practical application of hashing algorithms and file integrity monitoring, you'll learn how to construct a file integrity monitor. This hands-on approach will not only help you grasp the concept of 'Integrity', which is one of the three main triads of Cyber Security, but also empower you to implement these principles effectively."

  


<h2> Environments & Utilities Used: </h2>


- <b> Windows OS </b>
- <b> PowerShell ISE</b>
- <b> Notepad </b>
  


<h2>Project walk-through:</h2>

- <b>1. Defining Functions: </b> 

   - `Calculate-File-Hash`: This function calculates the SHA-512 hash of a given file and returns the hash value.

   - `Erase-Baseline-If-Already-Exists`: This function checks if the baseline file ("baseline.txt") exists and deletes it if it does.

   - `Collect-New-Baseline`: This function collects a new baseline by calculating hashes for files in the "Files" directory and exporting the information to a CSV file.

   - `Begin-Monitoring`: This function begins the monitoring process by comparing current files with the baseline data and notifying about changes.

- <b> 2. Main Script: </b>

   - The script starts by displaying options to the user using `Write-Host`.

   - It uses `Read-Host` to get the user's choice of action (A or B).

- <b> 3. Collecting New Baseline (Option A):</b>

   - If the user chooses option A:

     - The script pulls up `Collect-New-Baseline` function.

     - Within this function, `Erase-Baseline-If-Already-Exists` removes any existing baseline file.

     - It uses `Get-ChildItem` to retrieve a list of files in the "Files" directory.

     - For each file, it calculates the hash using `Calculate-File-Hash` function and creates a custom object with file path and hash.

     - This data is exported to a file ("baseline.txt").

     - A success message is displayed.

- <b> 4. Monitoring Files (Option B): </b>

   - If the user chooses option B:

     - The script pulls up `Begin-Monitoring` function.

     - Within this function, it imports the baseline data from the file ("baseline.txt").

     - It enters a continuous loop:

       - Retrieves the list of current files using `Get-ChildItem`.

       - For each file, it searches for the corresponding baseline information.

       - If there's no baseline info, it means the file is new, and it notifies accordingly.

       - If baseline info exists, it compares the current hash with the baseline hash. If they differ, it indicates the file has changed.

       - It also checks for deleted files by comparing baseline paths with actual paths.

       - Notifications are displayed using `Write-Host`.





  Function Calculate-File-Hash($filepath) {
    $filehash = Get-FileHash -Path $filepath -Algorithm SHA512
    return $filehash
}
Function Erase-Baseline-If-Already-Exists() {
    $baselineExists = Test-Path -Path .\baseline.txt

    if ($baselineExists) {
        # Delete it
        Remove-Item -Path .\baseline.txt
    }
}


Write-Host ""
Write-Host "What would you like to do?"
Write-Host ""
Write-Host "    A) Collect new Baseline?"
Write-Host "    B) Begin monitoring files with saved Baseline?"
Write-Host ""
$response = Read-Host -Prompt "Please enter 'A' or 'B'"
Write-Host ""

if ($response -eq "A".ToUpper()) {
    # Delete baseline.txt if it already exists
    Erase-Baseline-If-Already-Exists

    # Calculate Hash from the target files and store in baseline.txt
    # Collect all files in the target folder
    $files = Get-ChildItem -Path .\Files

    # For each file, calculate the hash, and write to baseline.txt
    foreach ($f in $files) {
        $hash = Calculate-File-Hash $f.FullName
        "$($hash.Path)|$($hash.Hash)" | Out-File -FilePath .\baseline.txt -Append
    }
    
}

elseif ($response -eq "B".ToUpper()) {
    
    $fileHashDictionary = @{}

    # Load file|hash from baseline.txt and store them in a dictionary
    $filePathsAndHashes = Get-Content -Path .\baseline.txt
    
    foreach ($f in $filePathsAndHashes) {
         $fileHashDictionary.add($f.Split("|")[0],$f.Split("|")[1])
    }

    # Begin (continuously) monitoring files with saved Baseline
    while ($true) {
        Start-Sleep -Seconds 1
        
        $files = Get-ChildItem -Path .\Files

        # For each file, calculate the hash, and write to baseline.txt
        foreach ($f in $files) {
            $hash = Calculate-File-Hash $f.FullName
            #"$($hash.Path)|$($hash.Hash)" | Out-File -FilePath .\baseline.txt -Append

            # Notify if a new file has been created
            if ($fileHashDictionary[$hash.Path] -eq $null) {
                # A new file has been created!
                Write-Host "$($hash.Path) has been created!" -ForegroundColor Green
            }
            else {

                # Notify if a new file has been changed
                if ($fileHashDictionary[$hash.Path] -eq $hash.Hash) {
                    # The file has not changed
                }
                else {
                    # File file has been compromised!, notify the user
                    Write-Host "$($hash.Path) has changed!!!" -ForegroundColor Yellow
                }
            }
        }

        foreach ($key in $fileHashDictionary.Keys) {
            $baselineFileStillExists = Test-Path -Path $key
            if (-Not $baselineFileStillExists) {
                # One of the baseline files must have been deleted, notify the user
                Write-Host "$($key) has been deleted!" -ForegroundColor DarkRed -BackgroundColor Gray
            }
        }
    }
}
