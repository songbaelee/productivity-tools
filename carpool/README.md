# Carpool Planner

A single page that shows a carpool's season: who drives there and who drives back each week, the next ride with a countdown, and a fairness tally with a previewed "Rebalance upcoming rides". Parents open one link and can view and edit without logging in.

- Page: `index.html` (plain HTML and JavaScript, no build step)
- Database: Cloud Firestore, through the Firebase Web SDK loaded from `gstatic.com`
- Config: `firebase-config.js`
- Security rules: `firestore.rules`

## How access works

Each carpool has a long random id (`c_` plus 24 random characters). The id is in the link after the `#`:

```
https://songbaelee.github.io/productivity-tools/carpool/#c_XXXXXXXXXXXXXXXXXXXXXXXX
```

Anyone with the link can view and edit everything in that carpool, including kids' names and pickup addresses. The security rules don't allow listing carpools, so nobody can find a carpool without its link. Treat the link like a shared password: send it only to the carpool's parents. The part after `#` is never sent to GitHub's servers.

Opening the page with no `#...` shows "Open your carpool's link, or create a new carpool."

## Data

```
carpools/{carpoolId}            { name, location, startTime, endTime,
                                  parents: [{id, name, active}],
                                  kids: [{id, name, address}], createdAt }
carpools/{carpoolId}/rides/{YYYY-MM-DD}
                                { date, to, home, cancelled, note, updatedAt }
```

`to` and `home` are parent ids, and are empty strings on cancelled days. Times and dates are stored as strings.

## Firebase setup (already done for this project)

1. In the [Firebase console](https://console.firebase.google.com/), create a project.
2. Build → Firestore Database → Create database, in **Production mode**.
3. Project settings → General → Your apps → add a **Web** app. Copy its `firebaseConfig` values into `firebase-config.js`. These values identify the project and are safe to publish; the rules control access.

## Deploying the security rules

1. Firebase console → Firestore Database → **Rules**.
2. Replace everything there with the contents of `firestore.rules`.
3. Click **Publish**. The new rules take effect within a minute.

Do this again whenever `firestore.rules` changes. (With the Firebase CLI installed you could run `firebase deploy --only firestore:rules` instead, but this project doesn't need the CLI.)

What the rules allow:

- `carpools/{id}`: anyone can read or update one carpool by its id. Creating needs an id of at least 20 characters. Listing and deleting carpools are denied.
- `carpools/{id}/rides/{date}`: anyone can read and write. Writes need the carpool to exist, the document id to be the ride's `date`, and the expected field types. Notes are limited to 500 characters.
- Everything else is denied.

## Recommended: restrict the API key

The API key in `firebase-config.js` is public. It's not a password, but limiting where it can be used stops other websites from using your project's quota.

1. Open [Google Cloud console → APIs & Services → Credentials](https://console.cloud.google.com/apis/credentials) and pick the `productivity-tools-c40af` project.
2. Open the key named **Browser key (auto created by Firebase)**.
3. Under **Application restrictions**, choose **Websites** and add:
   - `https://songbaelee.github.io/*`
   - `http://localhost:8000/*` (for testing locally; remove it if you never test locally)
4. Save. It can take a few minutes to apply.

## Making a new carpool link

1. Open `https://songbaelee.github.io/productivity-tools/carpool/` (or **New carpool** from any carpool's page).
2. Click **New carpool**. Enter the name, location, times, the parents (one per line) and the first and last dates. Dates repeat weekly and drivers are rotated evenly.
3. Click **Create**. The page moves to the new carpool's link.
4. Click **Copy link** and send it to the other parents. Bookmark it yourself.

There's no list of carpools anywhere, so keep the link. If it's lost, the carpool can only be found in the Firebase console (Firestore Database → Data → `carpools`).

## GitHub Pages

Repo Settings → Pages → Build and deployment → Source: **Deploy from a branch**, Branch: **main**, folder **/ (root)**. The page is then at `https://songbaelee.github.io/productivity-tools/carpool/`.

## Running locally

ES modules don't load from `file://`, so serve the repo folder:

```
cd productivity-tools
py -m http.server 8000
```

(On this PC `py` works; the plain `python` command may open the Microsoft Store instead.)

Then open `http://localhost:8000/carpool/#<carpool id>`. This uses the real Firestore database, not a copy.

## Deleting a carpool

The page can't delete carpools. In the Firebase console → Firestore Database → Data, select the carpool's document and delete it, including its `rides` subcollection.
