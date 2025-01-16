# Finvu Android SDK Integration Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Installation](#installation)
4. [Accessing Finvu SDK APIs](#accessing-finvu-sdk-apis)
5. [Initialization](#initialization)
6. [Usage](#usage)
7. [APIs](#apis)
8. [Frequently Asked Questions](#frequently-asked-questions)

## Introduction
Welcome to the integration guide for Finvu Android SDK! This document provides detailed instructions on integrating our SDK into your android application.

## Prerequisites
    1. Min SDK version supported is 24
    2. Min kotlin version supported is 1.9.0

## Installation

1. On android add the following repository to your project level `build.gradle` file. Github packages require that all packges need to be accessed with credentials. Our repository is public, so can be accessed with any github user. Create a  [PAT](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) which can be used here as GITHUB_PACKAGE_TOKEN
```
allprojects {
    repositories {
        google()
        mavenCentral()
        
        // Add these lines
        maven { 
            url 'https://maven.pkg.github.com/Cookiejar-technologies/finvu_android_sdk' 
            credentials {
                username = System.getenv("GITHUB_PACKAGE_USERNAME")
                password = System.getenv("GITHUB_PACKAGE_TOKEN")
            }
        }
    }
}
```
3. On android add the below in app level build.gradle file
```
    defaultConfig {
        minSdkVersion 24
    }

```

## Accessing Finvu SDK APIs
`FinvuManager` class that should be used to access the APIs on the SDK. `FinvuManager` class is a singleton, and can be access as follows:
```kotlin
    private val finvuManager = FinvuManager.shared
```

## Initialization
Initialize the SDK in your application's entry point (eg. splash screen). SDK can be initialized using the the following method.
```kotlin
    private val baseUrl = "wss://webvwdev.finvu.in/api"
    private val finvuClientConfig = FinvuClientConfig(finvuEndpoint = baseUrl))
```

## Usage
Refer to the SDK documentation for detailed instructions on using various features and functionalities provided by the SDK. Below is the sequence diagram which includes SDK initialization, account linking and data fetch flows.
![sequence-diagram](docs/Sequence-diagram.png)

## APIs

1. Initialize with config
Initialize API allows you to configure the finvuEndpoint. This is a way to configure the SDK to point to specific environments. 
```kotlin
        finvuManager.initializeWith(finvuClientConfig)
```

2. Connect 
Finvu exposes websocket APIs that the sdk interacts with. Before making any other calls, connect method should be called.
```kotlin
    finvuManager.connect { result ->
            runOnUiThread {
                if (result.isSuccess) {
                    Toast.makeText(this, "Connected", Toast.LENGTH_SHORT).show()
                } else {
                    when ((result.exceptionOrNull() as FinvuException).code) {
                        // messaging to the user
                    }
                }
            }
        }
```

3. Login with Consent Handle
Once the consent handle has been retrieved from the FIU server and we want to initiate the login flow for the user, it can be done in the following manner
```kotlin
FinvuManager.shared.loginWithUsernameOrMobileNumber(
      username = username,
      mobileNumber = mobileNumber,
      consentHandleId = consentHandleId
    ) {
      if (it.isFailure) {
        // handle errors 
      }

      val response = it.getOrThrow()
      val loginOtpReference = response.reference
    }
```

4. Verify login otp
The login response contains the otp reference. Once user enters the otp, verification can be triggered in the following way - 
```kotlin
FinvuManager.shared.verifyLoginOtp(otp = otp, otpReference = otpReference) {
      if (it.isFailure) {
        val error = it.exceptionOrNull() as FinvuException
        // understand the failure
      }

      val handleInfo = response.userId
    }
```
Post login success, sdk will keep this session authenticated for the user and the rest of the methods can be triggered.

5. Fetch all FIP options
Use this method to get all FIP details where discovery flows can be triggered
```kotlin
    FinvuManager.shared.fipsAllFIPOptions { result ->
      if (result.isFailure) {
        val error = result.exceptionOrNull() as FinvuException
        //handle the error
      }

      val response = result.getOrThrow()
    }
```

6. Account discovery
In order to initiate discovery flow, you will need to get FIP details first. Get them for the selected FIP in the following way
```kotlin
FinvuManager.shared.fetchFipDetails(fipId) { result ->
      if (result.isFailure) {
        val error = result.exceptionOrNull() as FinvuException
      }

      val response = result.getOrThrow()
    }
```
Once FIPDetails are available, discovery can be made with the following step - 

```kotlin
FinvuManager.shared.discoverAccounts(finvuFipDetails, fiTypes, finvuIdentifiers) { result ->
      if (result.isFailure) {
        val error = result.exceptionOrNull() as FinvuException
      }

      val response = result.getOrThrow()
    }
```
FITypes describe the type of accounts that we want to discover, typeIdentifiers provides the identifiers to discover them with. Here's an example of what thay may look like - 
```json
    {
        "category": "STRONG",
        "type": "MOBILE",
        "value": "9309107496"
    }
```

7. Initiate account linking
Once accounts have been discovered, linking flow may be initiated. Multiple accounts can be linked at once. 
```kotlin
FinvuManager.shared.linkAccounts(finvuAccounts, finvuFipDetails) { result ->
      if (result.isFailure) {
        val error = result.exceptionOrNull() as FinvuException
        // handle error
      }

      val response = result.getOrThrow()
      val accountLinkingRequestReference = (response.referenceNumber)
    }
```

8. Confirm account linking
On initiating account linking flow, an otp will be triggered by the FIP to the user. SDK will return a reference for this linking. Once user enters the top, linking can be confirmed by doing the following -

```kotlin
FinvuManager.shared.confirmAccountLinking(referenceNumber, otp) { result ->
      if (result.isFailure) {
        val error = result.exceptionOrNull() as FinvuException
      }

      val response = result.getOrThrow()
    }
```

9. Fetch all linked accounts
All existing linked accounts for the user can be fetched in the following manner -
```kotlin
    FinvuManager.shared.fetchLinkedAccounts { result ->
      if (result.isFailure) {
        val error = result.exceptionOrNull() as FinvuException
      }
      val response = result.getOrThrow()
    }
```

10. Consent flow
Once consent info is displayed to the user and user approves it, you can call the following method to convey the approval - 
```kotlin
FinvuManager.shared.approveConsentRequest(consentDetails, finvuLinkedAccounts) { result ->
      if (result.isFailure) {
        val error = result.exceptionOrNull() as FinvuException
      }

      val response = result.getOrThrow()
    }
```

In case user denies the consent, call this method instead - 
```kotlin
    FinvuManager.shared.denyConsentRequest(consentRequestDetailInfo);
```

## Frequently Asked Questions
Q. On Android I am getting the error `Class 'com.finvu.android.publicInterface.xxxxx' was compiled with an incompatible version of Kotlin. The binary version of its metadata is 1.9.0, expected version is 1.7.1.` or similar. How do I fix it?

A. Ensure that in your `settings.gradle` file, has the kotlin version set to 1.9.0
```
plugins {
    id "dev.flutter.flutter-plugin-loader" version "1.0.0"
    id "com.android.application" version "7.3.0" apply false
    id "org.jetbrains.kotlin.android" version "1.9.0" apply false <--- check version here
}
```
