import zipfile, os, shutil, tempfile, textwrap
src = "/mnt/data/srms-buph-full-package-1.zip" out = "/mnt/data/srms-buph-full-package-cloud-ci.zip"
workflow = r'''name: Build SRMS Android App (Cloud CI)
on: workflow_dispatch: push: paths: - "srms-android-app/" - ".github/workflows/android-build.yml" pull_request: paths: - "srms-android-app/" - ".github/workflows/android-build.yml"
permissions: contents: read
jobs: build-android: runs-on: ubuntu-latest
defaults:
  run:
    working-directory: srms-android-app

steps:
  - name: Checkout source
    uses: actions/checkout@v4

  - name: Set up JDK 17
    uses: actions/setup-java@v4
    with:
      distribution: temurin
      java-version: "17"

  - name: Set up Android SDK
    uses: android-actions/setup-android@v3

  - name: Install Android SDK packages
    run: |
      sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"
      yes | sdkmanager --licenses >/dev/null || true

  - name: Set up Gradle 8.6
    uses: gradle/actions/setup-gradle@v4
    with:
      gradle-version: "8.6"

  - name: Build debug APK
    run: gradle --no-daemon clean assembleDebug

  - name: Upload debug APK
    uses: actions/upload-artifact@v4
    with:
      name: srms-debug-apk
      path: srms-android-app/app/build/outputs/apk/debug/app-debug.apk
      if-no-files-found: error
      retention-days: 14
'''
cloud_readme = r'''# SRMS Android App — Cloud/CI Build
This Android project is configured to build the APK in GitHub Actions (cloud CI).
What changed
Added .github/workflows/android-build.yml.
The workflow uses Ubuntu in GitHub's cloud.
It installs/configures JDK 17, Android SDK 34, and Gradle 8.6.
It runs clean assembleDebug.
The resulting app-debug.apk is uploaded as a GitHub Actions artifact.
You do not need Android Studio installed on the computer that performs the cloud build.
You still need a GitHub repository to run the workflow.
How to build the APK in the cloud
Create/sign in to a GitHub account.
Create a new repository (for example, srms-buph).
Upload the contents of this package to the repository, keeping these folders:
srms-android-app/
.github/workflows/android-build.yml
srms-web-and-api/
Open the repository on GitHub.
Select Actions.
Choose Build SRMS Android App (Cloud CI).
Click Run workflow.
Wait until the workflow finishes with a green check.
Open the completed workflow run.
Under Artifacts, download srms-debug-apk.
Extract the downloaded artifact and install app-debug.apk on an Android phone.
Important
This workflow creates a debug APK for testing. It is not a Play Store release package and it is not signed with a personal release key.
The PHP/MySQL backend is still required for the app's real data. Cloud CI only builds the Android application; it does not automatically host the PHP/MySQL server.
For local development, Android Studio can still be used, but it is no longer required just to produce a test APK. '''
with zipfile.ZipFile(src, "r") as zin: with zipfile.ZipFile(out, "w", compression=zipfile.ZIP_DEFLATED) as zout: for item in zin.infolist(): data = zin.read(item.filename) zout.writestr(item, data)
zout.writestr(".github/workflows/android-build.yml", workflow)
    zout.writestr("srms-android-app/README-CLOUD-CI.md", cloud_readme)
print(f"Created: {out}") print(f"Size: {os.path.getsize(out)/1024:.1f} KB")
