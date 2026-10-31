name: Build NoorStone APK

on:
  workflow_dispatch:
  push:
    paths:
      - 'NoorStone-Android-JetpackCompose-Project.zip'
      - '.github/workflows/build.yml'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.9'

      - name: Extract project
        run: unzip -o NoorStone-Android-JetpackCompose-Project.zip -d source

      - name: Fix build configuration
        run: |
          cat > source/build.gradle.kts <<'EOF'
          plugins {
              id("com.android.application") version "8.7.3" apply false
              id("org.jetbrains.kotlin.android") version "2.0.21" apply false
              id("org.jetbrains.kotlin.plugin.compose") version "2.0.21" apply false
          }
          EOF

      - name: Fix missing manifest resources
        run: |
          python3 - <<'PY'
          from pathlib import Path
          p = Path("source/app/src/main/AndroidManifest.xml")
          s = p.read_text()
          s = s.replace('    package="com.noorstone.cartoon"', '')
          s = s.replace('        android:icon="@mipmap/ic_launcher"\n', '')
          s = s.replace('        android:roundIcon="@mipmap/ic_launcher_round"\n', '')
          s = s.replace('        android:label="@string/app_name"\n', '        android:label="Noor Stone"\n')
          s = s.replace('@style/Theme.NoorStoneCartoon', '@android:style/Theme.Material.Light.NoActionBar')
          p.write_text(s)
          PY

      - name: Build APK
        run: cd source && gradle assembleDebug

      - uses: actions/upload-artifact@v4
        with:
          name: NoorStone-APK
          path: source/app/build/outputs/apk/debug/*.apk
