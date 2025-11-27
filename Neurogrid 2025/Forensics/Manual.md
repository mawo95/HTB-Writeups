# The idea
The .bat file is heavily encoded so we want to let the program run and decode it for us and hten just grab the decoded output.

# Solution

Replace 
```iec($decoded);``` 
with 
```Write-Output $decoded``` 
so it prints the output into the console.

Also start the .bat with cmd /k x.bat to get the full output.

You also need to define a specific value which tells the program to actually start.
Replace 
```if not DEFINED hndswuidfAbc1hndswuidf (
    set hndswuidfAbc1hndswuidf=1   & cmd /c start  /min C:\Users\user\Downloads\SysinternalsSuite\tradesman_manual.bat    & exit
)
```
with 
```set hndswuidfAbc1hndswuidf=1
echo DEBUG: skipping self-relaunch
```

# Flag

We get this output: 
```
$anaba = Join-Path $env:USERPROFILE 'aoc.bat'
$uri    = 'http://malhq.htb/HTB{34dsy_d30bfusc4t10n_34sy_d3t3ct10n}'
```
Where we can see the flag!
