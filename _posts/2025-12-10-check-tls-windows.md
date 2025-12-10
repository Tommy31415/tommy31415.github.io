---
layout: post
title: Check if TLS 1.3 is enabled in your Windows 10
date: 2025-12-10
categories: ibm xms tls
---

## Overview

This script should check if TLS 1.3 is disabled or enabled in operating Windows system. It is important during negotiation cipher spec when connecting to IBM MQ using managed connection.

```powershell
# Define the registry path for TLS 1.3 Client configuration
$tls13ClientPath = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.3\Client"

Write-Host "Checking TLS 1.3 Client Configuration..." -ForegroundColor Cyan

# Check if the registry path exists
if (Test-Path $tls13ClientPath) {
    # Get the values for 'Enabled' and 'DisabledByDefault'
    $enabled = Get-ItemProperty -Path $tls13ClientPath -Name "Enabled" -ErrorAction SilentlyContinue
    $disabledByDefault = Get-ItemProperty -Path $tls13ClientPath -Name "DisabledByDefault" -ErrorAction SilentlyContinue

    # Display Enabled status
    if ($enabled.Enabled -eq 1) {
        Write-Host " [OK] TLS 1.3 is explicitly ENABLED (Enabled = 1)" -ForegroundColor Green
    } elseif ($enabled.Enabled -eq 0) {
        Write-Host " [!] TLS 1.3 is explicitly DISABLED (Enabled = 0)" -ForegroundColor Red
    } else {
        Write-Host " [?] 'Enabled' key is missing. Status depends on OS defaults." -ForegroundColor Yellow
    }

    # Display DisabledByDefault status
    if ($disabledByDefault.DisabledByDefault -eq 0) {
        Write-Host " [OK] TLS 1.3 is set to NOT be disabled by default (DisabledByDefault = 0)" -ForegroundColor Green
    } elseif ($disabledByDefault.DisabledByDefault -eq 1) {
        Write-Host " [!] TLS 1.3 is set to be DISABLED by default (DisabledByDefault = 1)" -ForegroundColor Red
    } else {
        Write-Host " [?] 'DisabledByDefault' key is missing." -ForegroundColor Yellow
    }
} else {
    Write-Host " [X] TLS 1.3 Registry Keys NOT FOUND." -ForegroundColor Red
    Write-Host "     On Windows 10, this usually means TLS 1.3 is DISABLED by default." -ForegroundColor Gray
}

Write-Host "`nNote: If keys are missing, Windows 10 default behavior is to disable TLS 1.3."

```








