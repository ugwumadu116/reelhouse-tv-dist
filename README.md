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

The push is enough: a GitHub webhook on this repository tells Coolify to rebuild, so a
new build is live a minute or so after `git push` with nothing to click. The hook posts to
`/webhooks/source/github/events/manual` on the Coolify host and is signed with the secret
stored under the application's **Webhooks** tab — the two have to match, so if the hook
ever starts returning 401, set a new secret in both places rather than clearing it in one.

If the hook is down, *Redeploy* on the application in Coolify does the same job by hand.

Installing a new build over the old one keeps resume positions and saved sources, as long
as it was signed with the same keystore.

The signing keystore lives with the app project and is **not** in this repository.
