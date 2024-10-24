# Passio SDK

[![release](https://img.shields.io/badge/release-v3.2.1-brightgreen)](https://github.com/Passiolife/Passio-Nutrition-AI-Android-SDK-Distribution/releases/tag/v3.2.1)    [![release](https://img.shields.io/badge/platform-Android-lightgray)]() [![release](https://img.shields.io/badge/minimum--suported--version-26-lightgray)](https://developer.android.com/about/versions/oreo)  [![release](https://img.shields.io/badge/Kotlin-v1.6.10-informational)](https://github.com/JetBrains/kotlin/releases/tag/v1.6.10) [![release](https://img.shields.io/badge/codelab-Get_started-important)](https://musing-gates-4e7160.netlify.app/#0)

## Overview:

Welcome to Passio Nutrition-AI Android SDK!

When integrated into your app the SDK provides you with food recognition and nutrition assistant technology. The SDK is designed to take a stream of images and output foods recognized in those images along with nutrition data related to the recognized foods.

As the developer, you have complete control of when to turn on/off the SDK and to configure the outputs which include:
- food names (e.g., Avocado toast, fruit salad, Clif bar)
- lists of related food in the hierarchy (children, siblings and parents) e.g. for milk: whole milk, soy milk, almond milk etc.  
- logos detected in the images
- lists of foods associated with the logos
- text detected on packages 
- branded foods matched to the text detected on food packages
- if nutrition labels are detected, the SDK returns nutrition associated with those labels
- UPC codes detected in the images
- nutrition information associated with the foods
- food amounts (those could be default amounts or amounts coming from our amount estimation module)

By default the SDK does not record/store any photos or videos. Instead, as the end user hovers over a food item with his/her camera phone, the SDK recognizes and identifies the food item in real time. This hovering action is only transitory/temporary while the end user is pointing the camera at a particular item and is never recorded or stored within the SDK. As the developer, you can configure the SDK to capture images or videos and store them in the main app.

### All of the documentation is provided on our [Gitbook page](https://passio.gitbook.io/nutrition-ai/guides/nutrition-ai-sdk)

### The UI Module implementation [source code and library](https://github.com/Passiolife/Passio-Nutrition-AI-Android-UI-Module)

<sup>Copyright 2024 Passio Inc</sup>