# Simple Diagnosis Script for: Network, Performance, & Storage    
# Author: Wade 

$ComputerName = $env:COMPUTERNAME 
$TimeStamp = Get-Date -Format "yyyyMMdd_HHmmss"
$Logpath = "$env:USERPROFILE\Desktop\Basic_Diagnostics_$ComputerName_$TimeStamp.txt" 

Write-Host "'nRunning basic diagnostics...'n"

#-----------------------------------------------
# NETWORK DIAGNOSTICS
#-----------------------------------------------
Write-Host "NETWORK CHECKS"

# Flush DNS
Write-Host "'nFllushing DNS cache..." 
ipconfig /flushdns 

# Restarting Network Adaptes(s)
Write-Host "'nRestarting network adapters...."
Get-NetAdapter | Restart-NetAdapter -Confirm:$false

# Display IP Configuration
Write-Host "'nCurrent IP Configuration:" 
ipconfig /all 

# Ping Google + Gateway
Write-Host "'nPinging Internet (Google)..."
Test-Connection -ComputerName 8.8.8.8 -Count 4

Write-Host "'nPinging default gateway..."
$gateway = (Get-NetRoute - DestinationPrefix "0.0.0.0/0").NextHop
if ($gateway) { 
  Test-Connection -ComputerName $gateway -Count 4 
  } else { 
    Write-Host "No default gateway detected." 
  } 

# ------------------------------------------------
#  PERFORMANCE DIAGNOSTICS 
# ------------------------------------------------
# Write-Host "'nPERFORMANCE CHECKS" 

# CPU & RAM Usage
$cpuUsage = (Get-Counter '\Processor(_Total)\%Processor Time').CounterSamples.CookedValue
$ram = Get-CimInstance -ClassName Win32_OperatingSystem
$freeMem = [math]::Round($ram.FreePhysicalMemory / 1MB, 2) 
$totalMem = [math]::Round($ram.TotalVisibleMemorySize / 1MB, 2) 
$usedMem = [math]::Round($totalMem - $freeMem, 2) 
$memPercent = [math]::Round((#usedMem / $totalMem) * 100, 1) 

Write-Host "'nCPU Usage: $([math]::Round($cpuUasage, 2))%"
Write-Host "RAM Usage: $usedMem GB / $totalMem GB ($memPercent%)" 

# Top Resource-Consuming Processes
Write-Host "'nTop 10 CPU-Consuming Processes:"
Get-Process | Sort CPU -Decending | Select -First 10 | Format-Table Name, CPU, -AutoSize 

Write-Host "'nTop Top Memory-Consuming Processes:" 
Get-Process | Sort WorkingSet -Decending | Select -First 10 | Format-Table Name, @{Name="Memory(MB)";Expression={[math]::Round($_.WorkinSet / 1MB, 1)}}, ID -AutoSize



















