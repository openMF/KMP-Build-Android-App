# KMP Build Android App Action

This GitHub Action automates the process of building Android applications using Gradle. It supports both Debug and Release builds, with optional signing configuration for Release builds.

## Features

- Debug and Release build support
- APK signing for Release builds
- Gradle dependency caching
- Artifact upload
- Google Services integration
- Fastlane automation

## Prerequisites

Before using this action, ensure you have:

1. A valid Android project with a Gradle build configuration
2. For Release builds:
  - A keystore file for signing your APK
  - Google Services JSON file (if using Firebase)

## Setup

### Fastlane Setup

Create a `Gemfile` in your project root:

```ruby
source "https://rubygems.org"

gem "fastlane"
```

Create a `fastlane/Fastfile` with the following content:

```ruby
default_platform(:android)

platform :android do
  desc "Assemble Debug APKs"
  lane :assembleDebugApks do
    gradle(
      task: "assemble",
      build_type: "Debug"
    )
  end

  desc "Assemble Release APKs"
  lane :assembleReleaseApks do |options|
    gradle(
      task: "assemble",
      build_type: "Release",
      properties: {
        "android.injected.signing.store.file" => options[:storeFile],
        "android.injected.signing.store.password" => options[:storePassword],
        "android.injected.signing.key.alias" => options[:keyAlias],
        "android.injected.signing.key.password" => options[:keyPassword],
      }
    )
  end
end
```

## Usage

Add the following workflow to your GitHub Actions:

```yaml
name: Build Android App

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      
      # For Debug Build
      - name: Build Debug APK
        uses: openMF/kmp-build-android-action@v2.0.0
        with:
          android_package_name: 'app'
          build_type: 'Debug'

      # For Release Build
      - name: Build Release APK
        uses: openMF/kmp-build-android-action@v2.0.0
        with:
          android_package_name: 'app'
          build_type: 'Release'
          google_services: ${{ secrets.GOOGLE_SERVICES }}
          keystore_file: ${{ secrets.RELEASE_KEYSTORE }}
          keystore_password: ${{ secrets.KEYSTORE_PASSWORD }}
          keystore_alias: ${{ secrets.KEYSTORE_ALIAS }}
          keystore_alias_password: ${{ secrets.KEYSTORE_ALIAS_PASSWORD }}
```

## Inputs

| Input                     | Description                                       | Required | Default |
|---------------------------|---------------------------------------------------|----------|---------|
| `android_package_name`    | Name of your Android project module (e.g., 'app') | Yes      | -       |
| `build_type`              | Build type to perform ('Debug' or 'Release')      | Yes      | 'Debug' |
| `google_services`         | Base64 encoded google-services.json file          | No       | -       |
| `keystore_file`           | Base64 encoded release keystore file              | No       | -       |
| `keystore_password`       | Password for the keystore file                    | No       | -       |
| `keystore_alias`          | Alias for the keystore file                       | No       | -       |
| `keystore_alias_password` | Password for the keystore alias                   | No       | -       |

## Outputs

| Output          | Description                                   |
|-----------------|-----------------------------------------------|
| `artifact-name` | Name of the uploaded artifact ('android-app') |

## Setting up Secrets

1. Encode your files to base64:
```bash
base64 -i path/to/release.keystore -o keystore.txt
base64 -i path/to/google-services.json -o google-services.txt
```

2. Add the following secrets to your GitHub repository:
- `RELEASE_KEYSTORE`: Content of keystore.txt
- `KEYSTORE_PASSWORD`: Your keystore password
- `KEYSTORE_ALIAS`: Your keystore alias
- `KEYSTORE_ALIAS_PASSWORD`: Your keystore alias password
- `GOOGLE_SERVICES`: Content of google-services.txt (if using Firebase)

## Artifacts

The action uploads the built APKs as artifacts with the name 'android-app'. You can find the APKs in:
- `build/outputs/apk/demo/`
- `build/outputs/apk/prod/`
