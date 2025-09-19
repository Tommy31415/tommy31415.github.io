---
layout: post
title: Configure IBM XMS managed connection
date: 2025-09-04
categories: ibm xms
---

## Overview

Export the personal certificate (with private key) from the CMS .kdb into a PKCS#12 (.p12/.pfx), import it into the Windows certificate store, and import the CA chain into Trusted Root/Intermediate.

Configure XMS for a managed connection with WMQ_SSL_KEY_REPOSITORY set to "*SYSTEM" or "*USER", set the CipherSpec to ECDHE_RSA_AES_256_CBC_SHA384, and optionally set the certificate label.

## 1) Export from KDB to PKCS#12
Find the client cert label in the KDB:

runmqakm -cert -list -db "C:\path\to\key.kdb" -stashed

Export the personal certificate plus private key to PKCS#12:

runmqakm -cert -export -db "C:\path\to\key.kdb" -stashed -label "YourClientCertLabel" -target "C:\path\to\client.p12" -target_type pkcs12 -target_pw "StrongPfxPassword"

Export the signer/CA certificates (if not bundled in the PFX) to files:

runmqakm -cert -extract -db "C:\path\to\key.kdb" -stashed -label "CA_or_Intermediate_Label" -target "C:\path\to\ca.cer" -target_type cert

## 2) Import into Windows certificate store
Decide scope:

Using "*SYSTEM": import into Local Computer\Personal (Certificates).

Using "*USER": import into Current User\Personal (Certificates).

Import the client PFX:

certlm.msc (Local Computer) or certmgr.msc (Current User) → Personal → Certificates → All Tasks → Import → select client.p12 → enter PFX password → ensure “Mark this key as exportable” is chosen if needed → finish.

Verify it shows “You have a private key that corresponds to this certificate.”

Set the Friendly Name (acts as the “certificate label” for the managed client) as needed:

Right‑click certificate → Properties → Friendly name.

If not specifying a label in code, set Friendly name to ibmwebspheremq<username in lowercase> to match the default lookup.

Import the CA/Intermediate certificates:

Local Computer (or Current User) → Trusted Root Certification Authorities → Certificates → Import the root CA.

Local Computer (or Current User) → Intermediate Certification Authorities → Certificates → Import intermediates.

## 3) Ensure Windows TLS can use the suite
Confirm TLS 1.2 is enabled and the cipher is available: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384.

Check with PowerShell (Admin):

Get-TlsCipherSuite | Where-Object Name -eq "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384"

If domain policy enforces a custom cipher order, add/position this suite in “SSL Cipher Suite Order” via Group Policy (gpedit.msc or domain GPO), then reboot the machine.

## 4) Configure the queue manager channel
On the SVRCONN channel used by the client, set SSLCIPH(ECDHE_RSA_AES_256_CBC_SHA384).

Ensure the queue manager server certificate is RSA and the full chain is present in the queue manager’s KDB.

## 5) C# XMS (.NET managed) setup
Use "*SYSTEM" or "*USER" for the Windows store, set the CipherSpec, and optionally pin the server certificate DN via SSL_PEER_NAME.

If the client certificate label must be specified explicitly, set both the XMS property and the IBM MQ .NET environment label to be safe.

Example (C#):
```c#
using IBM.XMS; using IBM.WMQ;

var ff = XMSFactoryFactory.GetInstance(XMSC.CT_WMQ);

var cf = ff.CreateConnectionFactory();

cf.SetIntProperty(XMSC.WMQ_CONNECTION_MODE, XMSC.WMQ_CM_CLIENT); // managed

cf.SetStringProperty(XMSC.WMQ_HOST_NAME, "mq.host.example.com");

cf.SetIntProperty(XMSC.WMQ_PORT, 1414);

cf.SetStringProperty(XMSC.WMQ_CHANNEL, "APP.SVRCONN");

cf.SetStringProperty(XMSC.WMQ_QUEUE_MANAGER, "QM1");

cf.SetStringProperty(XMSC.WMQ_SSL_KEY_REPOSITORY, "*SYSTEM"); // or "*USER"

cf.SetStringProperty(XMSC.WMQ_SSL_CIPHER_SPEC, "ECDHE_RSA_AES_256_CBC_SHA384");

cf.SetStringProperty(XMSC.WMQ_SSL_PEER_NAME, "CN=qm1.example.com,OU=..."); // optional pin

cf.SetIntProperty(XMSC.WMQ_SSL_KEY_RESETCOUNT, 0); // optional

cf.SetBooleanProperty(XMSC.WMQ_SSL_CERT_REVOCATION_CHECK, true); // optional

// Option A: XMS property for client cert label (if supported in your build)

cf.SetStringProperty(XMSC.WMQ_SSL_CLIENT_CERT_LABEL, "MyClientCertLabel"); // optional

// Option B: Also set the underlying IBM MQ .NET label to ensure selection

MQEnvironment.CertificateLabel = "MyClientCertLabel"; // optional

var conn = cf.CreateConnection();

var sess = conn.CreateSession(false, AcknowledgeMode.AutoAcknowledge);

var dest = sess.CreateQueue("queue:///APP.INPUT");

var prod = sess.CreateProducer(dest);

conn.Start();
```

### Notes:

For "*SYSTEM", the certificate must be in Local Computer\Personal; for "*USER", it must be in Current User\Personal.

If not setting a label, rename the certificate Friendly Name to ibmwebspheremq<windows_username_lowercase> so the managed client finds it by default.

SSL_PEER_NAME should match the server certificate’s Subject or SAN (typically the CN or a DNS SAN).

## 6) Quick validation and troubleshooting
If the connection fails with 2393 (SSL initialization), check:

The client certificate shows a private key and is in the correct store scope (SYSTEM vs USER).

The CA chain is present under Trusted Root/Intermediate.

The channel SSLCIPH matches the client CipherSpec exactly.

Windows offers TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 and TLS 1.2 is enabled.

If the wrong client certificate is used, explicitly set CertificateLabel (and/or adjust Friendly Name).

Review queue manager AMQERR01.LOG for messages like AMQ9631 (cipher mismatch) or AMQ9641/AMQ9633 (certificate chain issues).

## Diagrams

Here are two simple diagrams that illustrate the IBM MQ architecture and configuration discussed above:

![IBM MQ architecture](/assets/images/ibm-mq-xms/ibm-mq-architecture.svg)

![IBM MQ config overview](/assets/images/ibm-mq-xms/ibm-mq-config.svg)









