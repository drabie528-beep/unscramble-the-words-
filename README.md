# UNSCRAMBLE THE WORDS

A lightweight, responsive Friday activity built with plain HTML, CSS, and JavaScript. It works without a backend or build step.

## Run it

Open `index.html` in a current web browser. For local sharing across devices, serve this folder from a static web host (for example, GitHub Pages or Netlify) and share the resulting page link in WhatsApp. The Google Fonts stylesheet uses an internet connection; the system sans-serif fallback works offline.

## Play

Choose **PLAY NOW**, enter a name or nickname, and submit the registration form. Each of the five words has one answer submission and its own 15-second countdown, reset to 15 for each new word. Enter the answer and select **CHECK** (or press Enter). Every submission locks the answer and advances to the next word; a correct answer adds 20 points, while an incorrect answer adds none. An unanswered word is counted when its timer expires or an empty answer is submitted. The final result shows the player's name, score out of 100, correct answers out of 5, unanswered words, and total elapsed time. After all five rounds, the game reveals **START**. Select **SHOW ME THE MESSAGE** to see the English and Arabic encouragement. Games cannot be restarted after registration.

## Files

- `index.html` — screens, puzzle cards, final message, and CSS illustration
- `styles.css` — responsive visual design, transitions, and animations
- `game.js` — answer checking, solved-word tracking, progress, and screen flow

## Optional Google Sheets results

The game works without Google Sheets. To record each finished game's results, deploy the Apps Script below as a Web App and paste its `/exec` URL into `SHEETS_WEB_APP_URL` near the top of `game.js`. Each completed game submits the player name, score (20 points per correct word), correct answers, unanswered words, total time in seconds, and total word count once. Apps Script adds the submission date and time. Each game adds one row to a `Results` sheet tab, created automatically if it does not exist.

### Configure the spreadsheet

1. Create or open a Google spreadsheet.
2. Copy the spreadsheet ID from its URL. It is the text between `/d/` and `/edit` in `https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit`.
3. Open [script.google.com](https://script.google.com), create a new project, and replace the editor contents with this code. Replace `PASTE_SPREADSHEET_ID_HERE` with the ID from step 2.

```javascript
const SPREADSHEET_ID = 'PASTE_SPREADSHEET_ID_HERE';

function doPost(e) {
	const params = (e && e.parameter) || {};
	const playerName = String(params.playerName || '').trim();
	const score = Number(params.score);
	const correctAnswers = Number(params.correctAnswers);
	const unansweredWords = Number(params.unansweredWords);
	const totalWords = Number(params.totalWords);
	const totalTimeSeconds = Number(params.totalTimeSeconds);

	if (!playerName || !Number.isInteger(score) || !Number.isInteger(correctAnswers) ||
			!Number.isInteger(unansweredWords) || !Number.isInteger(totalWords) ||
			!Number.isInteger(totalTimeSeconds) || totalWords !== 5 ||
			score !== correctAnswers * 20 || score < 0 || score > 100 ||
			correctAnswers < 0 || correctAnswers > 5 || unansweredWords < 0 ||
			unansweredWords > 5 || correctAnswers + unansweredWords > 5 || totalTimeSeconds < 0) {
		return ContentService.createTextOutput('Invalid result');
	}

	const spreadsheet = SpreadsheetApp.openById(SPREADSHEET_ID);
	const sheet = spreadsheet.getSheetByName('Results') || spreadsheet.insertSheet('Results');
	if (sheet.getLastRow() === 0) {
		sheet.appendRow(['Submission Date/Time', 'Player Name', 'Score', 'Correct Answers', 'Unanswered Words', 'Total Time (seconds)', 'Total Words']);
	}
	sheet.appendRow([new Date(), playerName, score, correctAnswers, unansweredWords, totalTimeSeconds, totalWords]);
	return ContentService.createTextOutput('OK');
}
```

### Deploy the Web App

1. In the Apps Script editor, click **Deploy** > **New deployment**.
2. Click the gear icon, choose **Web app**, and set **Execute as** to **Me**.
3. Set **Who has access** to **Anyone**, then click **Deploy**. This access setting is needed so visitors to the static GitHub Pages game can submit results. The Web App URL is public, so anyone who has it can submit scores; connect only a sheet intended for public game results and do not store sensitive data in it.
4. Complete Google's authorization prompts. Copy the Web App URL ending in `/exec` from the deployment dialog.
5. In `game.js`, replace the empty string in `const SHEETS_WEB_APP_URL = '';` with that URL, keeping the quotes, for example: `const SHEETS_WEB_APP_URL = 'https://script.google.com/macros/s/DEPLOYMENT_ID/exec';`.
6. Save and publish the updated game files to GitHub Pages. After a game finishes, open the spreadsheet's `Results` tab to see appended rows. The browser uses a no-cors form submission, so it cannot display Google's response; the game remains playable if the URL is blank or Sheets is unavailable.

To update the script later, use **Deploy** > **Manage deployments**, edit the existing deployment, select a new version, and deploy it. Keep the same Web App URL in `game.js` unless you create a new deployment URL.
