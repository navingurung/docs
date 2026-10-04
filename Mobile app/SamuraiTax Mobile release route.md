SAMURAI TAX Mobile

# The release route, from Linear issue to store review

One line for the code, then it splits for iOS and Android and joins again at the tag. Follow the stations in order and never skip one.

Code and build Verify iOS Android

## Six rules

1. **Production builds come from `main` only**Never from staging or a feature branch.
2. **Build from a clean, pulled checkout**`git status` is clean, and Expo shows the commit without `*`.
3. **Commit the version bump before building**The version is baked into the build.
4. **Test every store build first**TestFlight for iOS, Internal testing for Android.
5. **Never edit build numbers**EAS increments them automatically.
6. **Tag every release**`vX.Y.Z` marks exactly what shipped.

## The route

1. R1

   ### Branch from staging

   Use the branch name from the Linear issue.

   ```
   git switch staging && git pull
   git switch -c <branch-name-from-linear>
   ```
2. R2

   ### Make the change and run it

   Test on a real iPhone. After a native change, run `npx expo prebuild --clean --platform ios` first and re-select the TAIMATSU signing team in Xcode.

   ```
   npx expo run:ios --device
   ```
3. R3

   ### Lint and test

   Zero lint errors and every test passing. Warnings don't block.

   ```
   npx expo lint && npm test
   ```
4. R4

   ### Bump the version

   Set `version: "X.Y.Z"` in `app.config.ts` and commit it on its own as `chore: bump version to X.Y.Z`.
5. R5

   ### Review into staging

   Push, open a PR into `staging`, get it approved, merge.
6. R6

   ### Promote to main

   Open a PR from `staging` into `main` and merge. `main` must equal what ships.
7. R7

   ### Get a clean checkout

   The last line must say “working tree clean”.

   ```
   git switch main && git pull && git status
   ```
8. R8

   ### Build in the cloud

   EAS builds iOS and Android together. No Android Studio or Xcode needed.

   ```
   npx eas build --profile production --platform all
   ```
9. V

   ### Verify on expo.dev

   Open Builds and check both new builds before going further.
   - Finished
   - Version X.Y.Z
   - Profile production
   - Commit without *

#### iOS line

1. I1

   ### Send to App Store Connect

   Uploads the latest iOS build from expo.dev.

   ```
   npx eas submit -p ios --latest
   ```
2. I2

   ### Test in TestFlight

   Wait for Apple to finish processing, install on an iPhone, run your checks.
3. I3

   ### Submit for review

   Distribution › new version X.Y.Z › select the build › What's New › Submit for Review.

#### Android line

1. A1

   ### Download the .aab

   expo.dev › Builds › the Android build › Download.
2. A2

   ### Test in Internal testing

   Play Console › Test and release › Testing › Internal testing › Create release. Upload, then install on the test phone with the opt-in link.
3. A3

   ### Send for review

   Promote the same release to Production, add release notes, send for review.

1. ### Tag the release

   Marks the exact code that shipped.

   ```
   git tag vX.Y.Z && git push origin vX.Y.Z
   ```

## Before you run the build

- [ ] Release PR merged into `main`
- [ ] On `main`, pulled
- [ ] `git status` says working tree clean
- [ ] `version` in `app.config.ts` is the new number
- [ ] Lint has 0 errors and tests pass
- [ ] Logged in to the company Expo account

## Good to know

Android signing key lives on EAS

expo.dev › Credentials › Android › `com.samuraitax`. Its SHA-1 matches Play Console's upload key. Never delete it.

`ios/` and `android/` are generated

Both are gitignored. EAS runs its own prebuild from `app.config.ts`, so local folders never reach store builds.

`eas build` uploads your local files

It never reads GitHub. A `*` on expo.dev means uncommitted changes went into the build.

Profile decides the app, branch decides the code

`production` makes the store app. `preview` builds from `staging` are for QA.