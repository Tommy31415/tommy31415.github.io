---
layout: post
title: Geneate certificate to sigin application
date: 2025-12-19
categories: pfx mage.exe
---

## Overview

This script should generate certificate to sign application using mage.exe

```powershell
# 1. Create a Self-Signed Code Signing Certificate in the "My" (Personal) store
$cert = New-SelfSignedCertificate -Subject "CN=MyClickOnceCert" `
                                  -CertStoreLocation "Cert:\CurrentUser\My" `
                                  -Type CodeSigningCert `
                                  -KeyUsage DigitalSignature `
                                  -FriendlyName "My ClickOnce Certificate"

# 2. Set a password for the PFX file (Mage requires this)
$password = ConvertTo-SecureString -String "MyStrongPassword123!" -Force -AsPlainText

# 3. Export the PFX file to your disk
$pfxPath = "C:\Temp\MyClickOnceCert.pfx"
Export-PfxCertificate -Cert $cert -FilePath $pfxPath -Password $password

Write-Host "Certificate created in store and PFX exported to: $pfxPath"
```








