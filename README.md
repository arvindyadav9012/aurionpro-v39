name: Build Android APK
on: [push, workflow_dispatch]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Extract Project
        run: |
          rm -rf project
          mkdir project
          FILE=$(find . -type f -name "*V39*READ*" | head -1)
          cp "$FILE" project/app.zip
          unzip -o project/app.zip -d project
      - name: Find gradlew
        run: |
          DIR=$(find project -name "gradlew" -type f | head -1 | xargs dirname)
          echo "GRADLE_DIR=$DIR" >> $GITHUB_ENV
          chmod +x $DIR/gradlew
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Add Location + Camera Permission
        run: |
          M=$(find ${{ env.GRADLE_DIR }} -name "AndroidManifest.xml" | head -1)
          sed -i '/<manifest/a \    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />\n    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />\n    <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />\n    <uses-permission android:name="android.permission.CAMERA" />' $M
      - name: Build APK
        run: |
          cd ${{ env.GRADLE_DIR }}
          ./gradlew assembleDebug
      - uses: actions/upload-artifact@v4
        with:
          name: Aurion-Fixed-APK
          path: ${{ env.GRADLE_DIR }}/app/build/outputs/apk/debug/*.apk
