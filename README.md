# Simple Diagnosis Script for: Network, Performance, & Storage    
# Author: Wade

$ComputerName = $env:COMPUTERNAME 
$TimeStamp = Get-Date -Format "yyyyMMdd_HHmmss"
$LogPath = "$env:USERPROFILE\Desktop\Basic_Diagnostics_${ComputerName}_$TimeStamp.txt"  

Start-Transcript -Path $LogPath -Append

Write-Host "`nRunning basic diagnostics...`n"

#-----------------------------------------------
# NETWORK DIAGNOSTICS
#-----------------------------------------------
Write-Host "NETWORK CHECKS"

# Flush DNS
Write-Host "`nFlushing DNS cache..." 
ipconfig /flushdns 

# Restarting Network Adapter(s)
Write-Host "`nRestarting network adapters..."
Get-NetAdapter | Restart-NetAdapter -Confirm:$false

# Display IP Configuration
Write-Host "`nCurrent IP Configuration:" 
ipconfig /all 

# Ping Google + Gateway
Write-Host "`nPinging Internet (Google)..."
Test-Connection -ComputerName 8.8.8.8 -Count 4

Write-Host "`nPinging default gateway..."
$gateway = (Get-NetRoute -DestinationPrefix "0.0.0.0/0").NextHop
if ($gateway) { 
  Test-Connection -ComputerName $gateway -Count 4 
} else { 
  Write-Host "No default gateway detected." 
} 

# ------------------------------------------------
#  PERFORMANCE DIAGNOSTICS 
# ------------------------------------------------
Write-Host "`nPERFORMANCE CHECKS"

# CPU & RAM Usage
$cpuUsage = (Get-Counter '\Processor(_Total)\% Processor Time').CounterSamples.CookedValue
$ram = Get-CimInstance -ClassName Win32_OperatingSystem
$freeMem = [math]::Round($ram.FreePhysicalMemory / 1MB, 2) 
$totalMem = [math]::Round($ram.TotalVisibleMemorySize / 1MB, 2) 
$usedMem = [math]::Round($totalMem - $freeMem, 2) 
$memPercent = [math]::Round(($usedMem / $totalMem) * 100, 1) 

Write-Host "`nCPU Usage: $([math]::Round($cpuUsage, 2))%"
Write-Host "RAM Usage: $usedMem GB / $totalMem GB ($memPercent%)" 

# Top Resource-Consuming Processes
Write-Host "`nTop 10 CPU-Consuming Processes:"
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 | Format-Table Name, CPU -AutoSize 

Write-Host "`nTop 10 Memory-Consuming Processes:" 
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 10 | Format-Table Name, @{Name="Memory(MB)";Expression={[math]::Round($_.WorkingSet / 1MB, 1)}}, ID -AutoSize

#-----------------------------------------------
# STORAGE DIAGNOSTICS
#-----------------------------------------------
Write-Host "`nSTORAGE CHECKS" 

# Drive Usage
Write-Host "`nDisk space on all drives:" 
Get-PSDrive -PSProvider 'FileSystem' |
Select Name, Used, Free, 
    @{Name='UsedGB';Expression={[math]::Round($_.Used/1GB,2)}}, 
    @{Name='FreeGB';Expression={[math]::Round($_.Free/1GB,2)}} |
Format-Table -AutoSize

# Disk Health (SMART status) 
Write-Host "`nDisk Health (SMART):"
Get-PhysicalDisk | Select DeviceID, MediaType, Size, SerialNumber, HealthStatus | Format-Table -AutoSize

# Done
Stop-Transcript
Write-Host "`nBasic diagnostics complete! Log saved to:`n$LogPath" 
Pause















