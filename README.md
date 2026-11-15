[README.md](https://github.com/user-attachments/files/34880721/README.md)
# Team Treasury (official version)

The secure, multi-team version of the DASC Parent Portal app. It has the same screens and features, rebuilt so the data is protected by real sign in and database security rules. The original app in **DASC Parent Portal App** is untouched and keeps running.

## What changed

| Before | Now |
|---|---|
| Everyone was signed in anonymously, and every visitor's browser downloaded the whole team: names, parent emails, access codes, PINs | Nothing loads until you sign in, and the database only sends what that person is allowed to see |
| Treasurer and coach PINs stored in the database | Google sign in (or an emailed link). Access is managed under **Settings → Team Access** |
| Parents used name + access code | Parents type their email and get a one-tap sign in link. It only works for an email on file for their player |
| Any transaction could be edited or deleted without a trace | **Void** (reason required) keeps the record. **Edit** voids the original and adds a corrected copy. The database itself blocks deletes |
| Archiving a season deleted the transactions | Archiving starts a fresh list, and old entries stay saved |
| "Reset everything" and "Restore" wiped the data | Removed. **Import a Backup** works only into an empty team (that's how you bring over the old data) |
| One team per Firebase project | Any number of teams. Each one sees only its own records |
| Treasurer notes visible to anyone | Notes stored separately, visible only to the treasurer and coach |

**Roles:** *Owner* (whoever creates the team) can do everything, including managing Team access. *Treasurer* can do everything except manage access. *Coach* has read-only access. The coach screen shows balances only, though their access technically includes the team's transactions and parent emails. *Parent* sees only their own player(s).

## Files

- `index.html`: the app (single file, no build step, same as before)
- `firestore.rules`: the database security rules
- `tests/`: automated security tests (optional, for development)

## Setup (about 20 minutes)

1. **Create the project.** At [console.firebase.google.com](https://console.firebase.google.com), click **Add project** and give it a name, for example `team-treasury`. Analytics isn't needed.
2. **Database.** Go to **Build → Firestore Database → Create database**, choose **production mode**, and pick a US location.
3. **Security rules.** In Firestore, open the **Rules** tab, replace everything with the contents of `firestore.rules`, and click **Publish**.
4. **Sign in methods.** Go to **Build → Authentication → Get started → Sign-in method**:
   - Enable **Google** and pick your support email.
   - Enable **Email/Password**, and inside it turn on **Email link (passwordless sign-in)**.
   - Do **not** enable Anonymous.
5. **Connect the app.** Under **Project settings → General → Your apps**, click the **</>** web icon, register the app, and copy the `firebaseConfig` values into the block near the top of `index.html` (replace the `PASTE-…` placeholders).
6. **Host it** on a new site so the old one stays as it is. For example, on Netlify use **Add new site → Deploy manually** and drag this folder in, or connect a new GitHub repo.
7. **Approve the web address.** In Firebase, go to **Authentication → Settings → Authorized domains** and add your new site's domain, for example `your-site.netlify.app`.
8. **First sign in.** Open the site, choose **Treasurer or Coach**, then **Continue with Google**. Create your team, and you become its owner.
9. **Bring over your data.** In the old app, go to **Settings → Download Full Backup**. In the new app, go to **Settings → Import a Backup** and choose that file. Then check a few balances against the old app.
10. **Add people.** Add the coach under **Settings → Team Access**. For parents, use **Roster → ✉ Send Invite**, which emails them the site link and how to sign in.

## Good to know

- Parents can only get in with an email that's on their player's record. If someone says "No players found," check the spelling of their email in the Roster.
- Sign in links expire after a while and work once. Parents stay signed in on that device afterwards.
- If the browser console ever shows a Firestore error with a link to "create an index," click the link and Firebase builds the index in a minute or two. None should be needed.
- The Firebase config in `index.html` isn't a secret. The rules in `firestore.rules` are what protect the data, so keep them published.

## Optional: Sign in with Apple

The button is built in but hidden until Apple is set up. You need a paid Apple Developer account ($99/year), which the App Store version needs anyway.

1. **Apple Developer → Certificates, Identifiers & Profiles → Identifiers:**
   - Create an **App ID** with *Sign in with Apple* checked.
   - Create a **Services ID**, for example `com.yourname.teamtreasury.web`. Turn on *Sign in with Apple*, click **Configure**, choose the App ID, and enter:
     - Domain: `my-parent-portal.firebaseapp.com`
     - Return URL: `https://my-parent-portal.firebaseapp.com/__/auth/handler`
2. **Keys:** Create a key with *Sign in with Apple* checked. Download the `.p8` file (you can only download it once) and note the **Key ID** and your **Team ID** (top right of the developer site).
3. **Firebase → Authentication → Sign-in method → Add new provider → Apple:** Enable it and fill in the Services ID, Team ID, Key ID and the contents of the `.p8` file.
4. **Optional but recommended:** Under **Services → Sign in with Apple for Email Communication**, register `noreply@my-parent-portal.firebaseapp.com` so emails can reach parents who use Hide My Email.
5. In `index.html`, change `window.ENABLE_APPLE_SIGNIN = false;` to `true`, then upload it to Netlify.

**Hide My Email:** Apple lets people hide their real address and give apps a relay address like `abc123@privaterelay.appleid.com`. That won't match the email on their player's record, so the sign in screen tells parents to choose **Share My Email**. If someone hides it anyway, the app explains what happened. You can also add their relay address as the player's second parent email.

## Running the tests (optional)

The tests need Node.js and Java. From the `tests` folder:

```
npm install
npx firebase emulators:exec --only firestore --project demo-tt "node rules.test.mjs"
```

This runs 48 checks against Firebase's official local emulator, such as "a parent can't see another family," "another team can't read or write yours," and "transactions can't be deleted."
