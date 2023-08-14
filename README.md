
<h2>Description</h2>

In this lab, I'll walk you through the process of setting up a file integrity monitor using live coding. By demonstrating the practical application of hashing algorithms and file integrity monitoring, you'll learn how to construct a file integrity monitor. This hands-on approach will not only help you grasp the concept of 'Integrity', which is one of the three main triads of Cyber Security, but also empower you to implement these principles effectively.

  


<h2> Environments & Utilities Used: </h2>


- <b> Windows OS </b>
- <b> PowerShell ISE</b>
- <b> Notepad </b>
  


<h2>Project walk-through:</h2>

**1. Open PowerShell:** 

- Ensure you have PowerShell installed on your system. 

- Go to your computer's Start menu. 

- Search for "PowerShell" and then select "PowerShell ISE" 

 



**2. Navigate to Script Directory:** 

- If you have the FIM script in a specific folder, navigate there using PowerShell. 

- Type `cd` followed by the path to your script directory and press Enter. 

  ``` 

  cd C:\Path\To\Your\Script\Directory 

  ``` 

  

**3. Clone Repository (If required):** 

- If the script is in a repository, clone it to your local machine. 

  

**4. Understand the Script:** 

- Open the script file (e.g., `script-name.ps1`) using Notepad or any text editor of your choice. 

- Examine the script to get an idea of its structure and functionality. 

  

**5. Hashes and Variables:** 

- The script calculates hash values using the `Calculate-File-Hash` function and stores them in variables. 

- Open the script and locate the `Calculate-File-Hash` function. Note how it computes the hash using SHA-512 and stores it in the variable `$filehash`. 

  

**6. Monitoring Files:** 

- In your FIM folder, you have files named a.txt, b.txt, c.txt, and d.txt. 

- Use the `Get-ChildItem` command to list the files in the FIM folder. Run the following command in PowerShell: 

  ``` 

  Get-ChildItem -Path .\FIM\Files\ 

  ``` 

  

**7. Baseline Data and Setup:** 

- Choose option A to collect a new baseline: 

  - Delete any existing baseline file by running the `Erase-Baseline-If-Already-Exists` function. 

  - Calculate the hash for each file and store the data in a "baseline.txt" file using the following command: 

    ``` 

    .\script-name.ps1 -Response A 

    ``` 

  

**8. Comparing Changes:** 

- Choose option B to begin monitoring: 

  - Load the baseline data from "baseline.txt": 

    ``` 

    .\script-name.ps1 -Response B 

    ``` 

  - The script enters continuous monitoring mode, calculating hashes and comparing them to the baseline. 

  

**9. Using Information for Change Detection:** 

- The script compares hashes to detect changes. If you suspect changes, run: 

  ``` 

  .\script-name.ps1 -Response B 

  ``` 

  

**10. Checking File Paths:** 

- To check file paths in the "Files" directory of the FIM folder, use: 

  ``` 

  Get-ChildItem -Path .\FIM\Files\ 

  ``` 

  

**11. Comparing Hashes:** 

- After running the script and detecting changes, verify hashes using: 

  ``` 

  .\script-name.ps1 -Response B 

  ``` 

  

**12. Configuring Baseline.txt:** 

- Set up the baseline.txt file within the FIM folder: 

  - Create a new text file named "baseline.txt" within the FIM folder. 

  - In the file, list file paths and their corresponding hash values.

    ``` 

    FIM\Files\a.txt,HashValue1 

    FIM\Files\b.txt,HashValue2 

 

**13. Notifications:** 

   - The script uses color-coded notifications for different scenarios: 

     - Green: New file detected. 

     - Yellow: File has changed. 

     - Dark Red on Gray: File has been deleted. 

 

# Functions & Script 
 
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

# Display menu options

Write-Host "File Integrity Monitoring Script"
Write-Host "================================"

Write-Host "What would you like to do?"
Write-Host "    A) Collect new Baseline?"
Write-Host "    B) Begin monitoring files with saved Baseline?"
Write-Host ""
$response = Read-Host -Prompt "Please enter 'A' or 'B'"
Write-Host ""


# Process user choice
if ($response -eq "A".ToUpper()) {
    # Delete baseline.txt if it already exists
    Erase-Baseline-If-Already-Exists


# Collect baseline data
    # Calculate Hash from the target files and store in baseline.txt
    # Collect all files in the target folder
    $files = Get-ChildItem -Path .\Files

    # For each file, calculate the hash, and write to baseline.txt
    foreach ($f in $files) {
        $hash = Calculate-File-Hash $f.FullName
        "$($hash.Path)|$($hash.Hash)" | Out-File -FilePath .\baseline.txt -Append
    }
    
}

# Set up dictionary
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
        $files

        # For each file, calculate the hash, and write to baseline.txt
        foreach ($f in $files) {
            $hash = Calculate-File-Hash $f.FullName
            #"$($hash.Path)|$($hash.Hash)" | Out-File -FilePath .\baseline.txt -Append
            
# Notifications 
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
 







