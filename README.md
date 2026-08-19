# Reel House — install host

Serves the Reel House Android TV APK at **https://tv.rivrafrica.com**, so the TV can
download it directly over the network instead of via a USB stick.

It is a static nginx image, deliberately separate from any other app on the server: a new
APK should never trigger a rebuild of something unrelated, and a mistake here should not
be able to take another site down.

## What it serves

| Path | Purpose |
| --- | --- |
| `/` | Install page — one large download button, plus instructions |
| `/a.apk` | The APK, under the shortest path a person can type on a D-pad keyboard |
| `/ReelHouse.apk` | The same file under its full name, for downloading on a laptop |
| `/healthz` | Returns `ok`, for the Coolify health check |

Both APK paths are served as `application/vnd.android.package-archive` with a
`Content-Disposition: attachment` header. Without that MIME type some TV browsers are
handed `application/octet-stream` and try to render the binary as text instead of saving
it.

## Publishing a new build

```bash
# from the app project
./gradlew :app:assembleRelease
cp app/build/outputs/apk/release/app-release.apk dist-site/site/ReelHouse.apk

cd dist-site
# update the version, size and sha256 shown on the page
shasum -a 256 site/ReelHouse.apk
git commit -am "Reel House <version>"
git push
```

Then redeploy the application in Coolify. Installing a new build over the old one keeps
resume positions and saved sources, as long as it was signed with the same keystore.

The signing keystore lives with the app project and is **not** in this repository.
