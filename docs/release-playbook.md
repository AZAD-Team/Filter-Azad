# Release Playbook

This playbook standardises how AZAD-Team prepares and publishes new Android
builds for the Filter Azad product line.

## 1. Build Production Artifacts

- Ensure the Flutter toolchain is on the approved LTS channel.
- Bump the `version` in `pubspec.yaml` and the Android `versionName` /
  `versionCode`.
- Run the release pipeline:

  ```bash
  flutter clean
  flutter pub get
  flutter build apk --split-per-abi --dart-define=ENV=production
  ```

- Collect the universal or ABI-specific APK produced under
  `build/app/outputs/flutter-apk/`.

## 2. Sign the APK

- Use the production keystore and alias documented in the secure vault.
- Sign the binary via:

  ```bash
  jarsigner -verbose -sigalg SHA256withRSA -digestalg SHA-256 \
    -keystore azad-production.keystore app-release-unsigned.apk prod_alias

  zipalign -v -p 4 app-release-unsigned.apk Azad_VX.Y.Z.apk
  apksigner sign --ks azad-production.keystore --out Azad_VX.Y.Z.apk Azad_VX.Y.Z.apk
  ```

## 3. Verify the Output

- Run instrumentation smoke tests on a physical device (API 24, 28, 33).
- Execute `apksigner verify --print-certs Azad_VX.Y.Z.apk` and confirm the
  certificate fingerprint matches the production key.
- Scan the build with the internal security suite before publication.

## 4. Update This Repository

- فایل‌های APK را در محیط محلی نگه دارید و آن‌ها را وارد کنترل نسخه نکنید.
- هش‌ها را پس از خروجی گرفتن محاسبه کنید:

  ```bash
  sha256sum Azad_VX.Y.Z.apk Filter_Azad_VX.Y(Beta).apk > checksums/sha256sum.txt
  ```

- بخش «یادداشت‌های انتشار» در `README.md` را با نسخه‌ها و نکات تازه به‌روزرسانی
  کنید.
- تغییرات را کامیت کنید:

  ```bash
  git add checksums/ README.md SECURITY.md
  git commit -m "chore(release): publish vx.y.z"
  ```

## 5. Publish & Announce

- Push to the `main` branch on GitHub.
- در GitHub یک Release جدید با تگ مناسب بسازید، APKهای پایدار و بتا را در بخش
  Assets بارگذاری کنید و لینک‌های دانلود را در متن Release و جدول README قرار
  دهید.
- Notify the community channel and support desk with the release highlights.

Maintain this checklist to keep the distribution process auditable and
repeatable.
