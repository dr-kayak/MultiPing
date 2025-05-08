$scriptPath = $PSScriptRoot
$deviceFile = Join-Path $scriptPath "devices.txt"
$logFile = Join-Path $scriptPath "ping_results.log"

# Clear previous log
if (Test-Path $logFile) { Clear-Content $logFile }

# Ask for input method
$useFile = Read-Host "Use devices.txt from script folder? (Y/N)"
$deviceList = @()

if ($useFile -match "^[Yy]$") {
    if (-not (Test-Path $deviceFile)) {
        Write-Host "devices.txt not found in script directory. Exiting."
        exit
    }
    $deviceList = Get-Content $deviceFile | Where-Object { $_.Trim() -ne "" }
} else {
    $manualInput = Read-Host "Enter hostnames/IPs separated by commas, semicolons, or spaces"
    $deviceList = $manualInput -split '[,; ]+' | Where-Object { $_.Trim() -ne "" }
}

# Choose ping mode
Write-Host "`nPing Mode Options:"
Write-Host "1 - Single ping"
Write-Host "2 - Continuous ping (until Ctrl+C)"
Write-Host "3 - Timed ping for X minutes"
$mode = Read-Host "Select mode (1/2/3)"

$durationMinutes = 0
if ($mode -eq "3") {
    $durationMinutes = Read-Host "Enter duration in minutes"
    if (-not [int]::TryParse($durationMinutes, [ref]$null) -or $durationMinutes -le 0) {
        Write-Host "Invalid duration. Exiting."
        exit
    }
}

# Initialize counters
$stats = @{}
foreach ($device in $deviceList) {
    $stats[$device] = [PSCustomObject]@{
        Success = 0
        Failure = 0
        Attempts = 0
    }
}

# Start time
$startTime = Get-Date

# Function: One round of pings
function Perform-Ping {
    foreach ($device in $deviceList) {
        $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
        $stats[$device].Attempts++
        try {
            $result = Test-Connection -ComputerName $device -Count 1 -Quiet -ErrorAction Stop
            if ($result) {
                $stats[$device].Success++
                $msg = "[$timestamp] $device is online"
            } else {
                $stats[$device].Failure++
                $msg = "[$timestamp] $device is offline"
            }
        } catch {
            $stats[$device].Failure++
            $msg = "[$timestamp] $device error: $_"
        }

        Write-Output $msg
        Add-Content $logFile -Value $msg
    }
}

# Run selected mode
if ($mode -eq "1") {
    Perform-Ping
}
elseif ($mode -eq "2") {
    Write-Host "`nPress Ctrl+C to stop..."
    while ($true) {
        Perform-Ping
        Start-Sleep -Seconds 5
    }
}
elseif ($mode -eq "3") {
    $endTime = (Get-Date).AddMinutes($durationMinutes)
    while ((Get-Date) -lt $endTime) {
        Perform-Ping
        Start-Sleep -Seconds 5
    }
} else {
    Write-Host "Invalid mode selected. Exiting."
    exit
}

# End time
$endTime = Get-Date
$duration = New-TimeSpan -Start $startTime -End $endTime

# Nicely formatted summary
Add-Content $logFile "`n--- SUMMARY ---"
Add-Content $logFile "Started: $($startTime.ToString('g'))"
Add-Content $logFile "Ended:   $($endTime.ToString('g'))"
Add-Content $logFile "Duration: $([math]::Round($duration.TotalMinutes, 2)) minutes"

foreach ($device in $deviceList) {
    $stat = $stats[$device]
    Add-Content $logFile "$device => Attempts: $($stat.Attempts), Success: $($stat.Success), Failure: $($stat.Failure)"
}

Write-Host "`nPing complete. See 'ping_results.log' for results." -ForegroundColor Green
