# Ubisoft Account Helper 1.3.4

A Windows application that prepares and submits one Ubisoft registration at a time. The interface is in English. MailSlurp can supply the email address, receive the new verification email and enter its code on Ubisoft's page.

## Changed in 1.3.4

Login now records actual trusted button/submit events and passive request/response counts. If the same valid form is still present after five seconds and no authentication-origin POST has been observed, the app can click Log in once more. Both attempts share the original 40-second result deadline and a two-click limit, including after Stop/Resume. Pending requests, any received response, changed credentials or controls, loading screens, CAPTCHA and visible errors prevent a retry. Response bodies, headers, passwords and tokens are never recorded.

The responsive account page may display Public profile before its separate Login information data is ready. While waiting for matching email and username, inspection now makes a bounded pass through the lower cards and reports which card or label is missing. The account page receives its own 90-second readiness window after login. Closure still requires both identity values to match the newly created account. Scrolling or a successful login alone never authorizes closure.

The specific cause of missing Login information in the reported live session has not been confirmed. These changes add bounded recovery and precise diagnostic reasons instead of treating every incomplete page as an unnamed loading state. Offline tests cover the responsive card structure, delayed card availability, mismatches, second-click recovery, and pending/failed/successful requests that forbid another login click. No live account closure was run during development.

## Fixed in 1.3.3

Ubisoft briefly covers the post-registration login form with a loading layer. Earlier versions immediately paused with `login-fields-obscured`. The app now waits and checks again on its existing timer, then fills the same account's credentials and clicks Log in once after the controls become available. The same wait handles temporarily disabled fields and a covered Log in button. No manual Start or fixed sleep is needed for a short loading transition.

The wait is limited to 45 seconds from the original login-page observation. It never enters covered fields, dismisses overlays or extends the deadline on each check. A persistent obstruction pauses with its reason code; Stop cancels the wait. Login errors, CAPTCHA and changes to account identity or credentials still pause immediately. No login is attempted after an account closure request.

Offline browser tests reproduce a delayed loading layer, disabled fields, a layer appearing after filling, cancellation during loading, and a persistent layer reaching the deadline. These checks do not constitute a live account login or closure.

## Changed in 1.3.2

The post-registration login moves offscreen fields into view immediately, with minimal scrolling, before checking that each field is unobstructed. The Log in button also uses immediate scrolling. This removes reliance on a smooth-scroll animation finishing during the same script call. Controls with hidden checkbox presentation cause a clear pause instead of leaving a persistent sign-in selected unnoticed.

Login diagnostics now record the exact stage, state and a fixed reason code. The status bar explains the failed check and includes that code, instead of only saying that the form could not be identified. These diagnostics contain no email address, password, mailbox/API credentials, page text or URL. Open diagnostics shows the existing local log.

The screenshot's exact failure was not reproduced on Ubisoft's public login page: both field hit tests succeeded there at the app's viewport size. This release tests decorated fields, smooth scrolling, embedded-frame scrolling and foreign overlays offline; it does not claim a verified live account login or closure. If another pause occurs, its status code identifies which check failed.

Open the **1.3.4** executable. An older running window keeps its current browser session and code; it does not update in place. Saved account records remain shared.

## Added in 1.3.1

After successful registration, Ubisoft may open its account-management login page with a separate embedded WebAuth form. The app now signs in there using the same newly created account's email and password, then resumes the existing account identity check and closure sequence. It supports the official embedded form and direct Ubisoft login pages, scrolls the Log in button into view, and sends one normal browser click. Keep me logged is turned off.

This login is only available before a closure request has been sent. Once closure is requested or confirmed, the app never automatically signs in again, because that could cancel the pending deletion. Stop cancels pending login actions. Login errors, CAPTCHA or an extra verification prompt pause the flow; credentials are not repeatedly submitted. Account history records a login attempt without storing its password in the text log.

An older running window keeps its existing session and code; use its page manually to finish that attempt, or start the updated app for a fresh run. Saved account records remain shared.

## Added in 1.3.0

**Close this account after registration** is enabled by default. After a successful new-account creation response is observed for the current run, the app opens [Account information](https://www.ubisoft.com/en-gb/account/account-information), checks that the profile matches the new account, scrolls to **Close account**, accepts the closure checkbox and presses **Continue** once.

When the visible closure dialog says **Your account will soon be closed**, the first observed timestamp is saved as `ClosureConfirmedAtUtc` in both the encrypted account store and the activity log. Saved accounts displays the local time with its UTC offset; Copy all details includes UTC too. This records Ubisoft's confirmation, not the date of permanent deletion. The app then clicks **Log out** in that same confirmation dialog and checks the result. A click alone is not recorded as a successful logout.

**Estimated deletion** is that saved confirmation plus exactly 30 days (720 hours), computed in UTC. The date, time to the second and UTC offset appear in Saved accounts. **Deletion timestamp** is the copyable Unix timestamp in seconds; Copy all details also includes ISO UTC. The activity log contains `EstimatedDeletionAtUtc` and `EstimatedDeletionUnixSeconds`. These values stay empty until closure is confirmed. They are estimates based on the displayed 30-day period, not a guarantee of Ubisoft's actual deletion time.

The account-creation response, run, private browser session, expected username and displayed email must agree before closure can proceed. Ubisoft normally masks the profile email, so the documented mask is allowed only with the same run's confirmed new-account creation. Email verification or a redirect alone cannot enable closure. If the account, dialog or result is unclear, the app pauses. It does not sign in again after closure or resend the destructive confirmation. If the timestamp cannot be saved, Log out is not clicked; Start can retry saving the same observation.

Uncheck the closure option before Start to keep a newly created account open. The choice locks with the registration attempt. No existing saved accounts are processed. No live closure was performed during development.

## Email matching fixed in 1.3.0

MailSlurp may return a short inbox address with capital letters while the delivered message's recipient is lowercase. Earlier versions compared these addresses case-sensitively and discarded the matching message, eventually reporting a timeout. Recipient matching now ignores letter case after checking the exact inbox ID and message ID. Sender, timestamp, code context and ambiguity checks remain in place.

The regression tests cover the mixed-case short address, lowercase recipient and full polling path, as well as genuinely different addresses and unrelated inboxes. The message format has not been broadened, and email addresses saved in existing account records are not rewritten.

## Changed in 1.2.3

- New MailSlurp inboxes request `useShortAddress: true`, using MailSlurp's [short email address option](https://www.mailslurp.com/guides/creating-inboxes/#short-email-addresses). Existing inbox addresses are kept as they are. Provider restrictions are reported; the app does not create a second fallback inbox.
- A denied initial registration request now displays **HTTP 403** and keeps watching the page for up to five minutes. Continue on Ubisoft's page; when its email-verification form appears, the same run and mailbox continue automatically. **Stop automation** cancels this monitoring.
- The 403 wait sends no additional account clicks and does not refill the form. Diagnostics classify a recognized security response, a numeric API error or an unknown response without saving its content.

The observed live failure was a delivered click followed by HTTP 403; the user's later manual click received HTTP 200 and opened email verification. The first denial's cause is not yet confirmed. This version fixes continuation after that manual advance; it does not claim to make a denied request succeed automatically.

## Fixed in 1.2.2

- Focuses the embedded browser and account button, allows focus/blur processing to settle, and checks Angular validation as well as ordinary field validity before clicking.
- Rechecks the same fields, required checkboxes and button immediately before dispatch. Visible `.invalid-feedback` messages are brought into view.
- Records whether the exact button received ordinary trusted pointer/click/submit events; a focus outline alone is not treated as a delivered click.
- Allows **one additional Create account click** after an unchanged valid form remains on screen for at least five seconds. It requires a known button receipt, no pending/error response and no observed email-code or actual account-creation request. Pending validation, loading screens, CAPTCHA, changed details or uncertain responses prevent the retry. The email-code submit step is not automatically retried.
- **Open diagnostics** opens `Logs/automation.jsonl` under the app's local data folder. It contains run IDs, stage names, counters, HTTP status and numeric validation codes. It contains no field values, email codes, request/response bodies, full URLs or API keys.

Open the **1.3.4** executable for these changes. An already running older window keeps its own browser session and old code; saved account records remain shared. If finishing an existing attempt in an older window, its code can still be entered manually. Start a fresh registration in the updated version to use the corrected email matching.

## Fixed in 1.2.1

- Recognizes Ubisoft's actual `/email-validation-enforcement` registration step, its six-digit **Verification code** field and final **Create account** button.
- Reads the **Ubisoft Account Security Code / Email Validation** email template and extracts the code from its expiry paragraph. Address and postal-code numbers in the footer are not treated as verification codes.
- Detects the form's `.invalid-feedback` errors instead of continuing after an invalid or expired code.

The form structure was checked against [Ubisoft's public application bundle](https://ubistatic-a.ubisoft.com/0176/prod/bundles/dist/app44c194a7683692a66711.bundle.js) and reproduced in the offline WebView2 tests. During the 1.2.1 work, a read-only MailSlurp inbox check successfully extracted the real message's code without recording it or sending it to Ubisoft. That historical report remains in the 1.2.1 release folder as `mailslurp-inbox-check.json`. No new live inbox check was run for 1.2.2 or 1.2.3, and a successful live account completion is still not claimed.

The earlier mail-verification fixes are included in 1.3.0. Older windows do not update themselves or transfer their in-progress browser sessions.

## Start with MailSlurp

1. Open `UbisoftAccountHelper.exe` and wait for the embedded browser to load.
2. Select **MailSlurp** under **Email provider**, then click **Setup**.
3. Enter the API key from your [MailSlurp dashboard](https://app.mailslurp.com/). Leave **Existing inbox ID** blank to create one inbox when you press Start, or enter an existing inbox ID to reuse that inbox. **Save settings** only updates this application's memory; it does not create an inbox.
4. Enter a username and password, or leave them blank to generate them. Review the date of birth, which defaults to **01/01/1990** in day/month/year format.
5. Click **Start automatic registration**. The app connects or creates the inbox, rejects optional cookies, opens registration, fills the date and account fields, accepts the required Ubisoft terms/privacy checkboxes and clicks the registration button. Newsletters remain unchecked.
6. If Ubisoft opens an identifiable email-code form, the app waits up to **2 minutes** for a new matching Ubisoft email, fills an unambiguous code and clicks the verification button. It then checks the page for a result.

The API key stays in memory for the current session and must be entered again after closing the app. It is sent only to MailSlurp's HTTPS API in an authentication header. The app does not save the key, verification code or email body in its account files or logs.

Short addresses apply to newly created inboxes. Reusing an existing inbox ID preserves its current address and messages.

If a mail request fails after registration, stop the run and open **Setup** to replace the API key. The existing inbox and account details remain locked; Start resumes the same attempt.

An inbox is created without a requested expiration. MailSlurp's plan may impose one; the app displays a returned expiration date. Inboxes are kept in your MailSlurp dashboard and are not deleted by the app. Check retention and keep access to the inbox for future account recovery.

If an inbox-creation request is cancelled or loses its connection, an inbox may still have been created on MailSlurp's side. Check the dashboard and enter its ID in **Setup** before trying to create another.

## Another account or a manual email address

Use **New account** after completing or stopping the current run. It clears this window's Ubisoft browser session and current form details. Saved accounts and MailSlurp inboxes remain available. The next Start creates a fresh inbox unless you connect an existing ID in Setup. Accounts run sequentially; there is no unattended batch queue.

Select **Manual email** to use an address you already own. The form automation still runs, but you complete email verification on Ubisoft's page yourself.

**Stop automation** pauses the active run. A CAPTCHA, uncertain page state, provider failure or unrecognized verification form also pauses automation with a status message. Complete a CAPTCHA on Ubisoft's page. The app does not blindly repeat a Create account or verification click after an uncertain result, and it does not automatically resend email codes. Inspect the page before deciding how to continue.

## Saved account details and the activity log

After a run starts successfully, account details are saved before the automated registration click. Attempts are retained even when registration fails or stops. **Saved accounts** shows the latest recorded status for each run and includes email, username, password, date of birth, provider and inbox ID. Select a row to view details. Passwords are masked until you choose **Show password**; **Copy password** and **Copy all details** deliberately place the selected information on the Windows clipboard.

The account store is encrypted with Windows DPAPI for the current Windows user:

```text
%LOCALAPPDATA%\UbisoftAccountHelper\Accounts\accounts.dpapi
```

Use the same Windows profile to open saved accounts. Copying this encrypted file alone to another computer or Windows account does not provide a portable password backup. If the file cannot be decrypted or is damaged, the app reports the problem and preserves it instead of overwriting it with an empty store.

**Open account log** opens the chronological metadata log in Notepad:

```text
%LOCALAPPDATA%\UbisoftAccountHelper\Logs\accounts.jsonl
```

The ordinary JSONL log contains timestamps, run IDs, email addresses, usernames, provider/inbox IDs and statuses. Passwords and birthdates are kept in the encrypted account store rather than this text log.

| Status | Meaning |
| --- | --- |
| `prepared`, `resumed` | Details were saved and the run started or resumed. |
| `registration_attempted` | A Create account click was attempted; this does not confirm account creation. |
| `waiting_for_email`, `verification_required` | Email verification is pending. |
| `verification_attempted` | The email-code verification click was attempted. |
| `email_verified` | Ubisoft's page explicitly indicated that the email was verified. |
| `account_created_confirmed` | Ubisoft explicitly confirmed creation in its page or the observed creation response. Automatic closure requires the current run's successful creation response. |
| `account_login_attempted` | The newly created account was submitted to the official login form before closure. |
| `account_identity_confirmed` | The account profile matches the new registration. |
| `closure_requested` | The closure confirmation was clicked; its result is pending. |
| `closure_confirmed` | The final closure confirmation appeared and its observation time was saved. |
| `logged_out` | Logout was observed after the saved closure confirmation. |
| `closure_unknown`, `closure_action_required` | Closure or logout needs review; no success is assumed. |
| `unknown_result` | The page did not establish a clear result. |
| `action_required` | Ubisoft denied the request; continue on its page. The app can keep watching for email verification. |
| `paused`, `stopped`, `error`, `mail_error`, `mail_timeout` | Work paused, stopped or encountered the indicated problem. |

Do not treat a saved row or an attempted click as proof of a completed account. Check its recorded status and Ubisoft's page.

## Version and requirements

The window title should show **1.3.4**. Previous versions remain in their own folders. Details held only in an older running version are not migrated into this version; save or copy those details before closing the older window.

The packaged release requires **Windows x64** and the **Microsoft Edge WebView2 Runtime**. The .NET runtime is included in the published application. Internet access and your own MailSlurp account/API key are needed for the live email workflow. MailSlurp plan limits and inbox retention still apply.

## Validation and source build

Version 1.3.2 extends the offline suite with decorated login fields, offscreen smooth scrolling, tall embedded frames, obstruction checks and privacy checks for login diagnostics. Version 1.3.1 added embedded login, cancellation, single-attempt limits and the no-login-after-closure guard. Version 1.3.0 added recipient normalization, new-account response attribution, closure identity and dialog guards, timestamp persistence, and native click/confirmation/logout tests. See this release's `test-results.json` for the recorded outcome.

The suite uses actual WebView2 forms for navigation, scrolling, input, clicks, bounded retries and verification-state handling; offline request responses for retry decisions; fake HTTP responses for MailSlurp filtering, timeouts and cancellation; and actual Windows DPAPI for encrypted-account roundtrips, updates, concurrent writes and damaged-file handling. See `test-results.json` for this release's recorded outcome.

The supplied test key passed a live MailSlurp API check in 1.2.0: one inbox was created, read back and polled for new mail. Version 1.2.1 additionally checked the existing inbox's real Ubisoft verification message. The key is not included in the release or test reports.

No real Ubisoft account was created by the implementation tests. A real Ubisoft email was observed in the inbox used by the user. Live end-to-end registration, closure and logout remain unverified; the form and code-entry workflow was exercised with offline fixtures.

To build from this source folder with the .NET 10 SDK installed:

```powershell
dotnet publish .\UbisoftAccountHelper.csproj -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true -p:IncludeNativeLibrariesForSelfExtract=true -p:DebugType=None -o .\publish
```

The executable also supports `--self-test --results <absolute-json-path>` for its offline suite. This does not create a real MailSlurp inbox or Ubisoft account.

MailSlurp references: [API endpoint specification](https://api.mailslurp.com/v2/api-docs/), [inbox creation and retention options](https://www.mailslurp.com/docs/inboxes/).

