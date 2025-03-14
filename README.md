# Submit HTML Form Data to Google Sheets Without PHP

This guide explains how to submit data from an HTML form directly to a Google Sheet without using PHP. By using Google Apps Script, you can easily capture form submissions and store them in a Google Sheet.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Step-by-Step Guide](#step-by-step-guide)
3. [HTML Form Setup](#html-form-setup)
4. [JavaScript Code](#javascript-code)

---

## Prerequisites

- A Google Account.
- Basic knowledge of HTML, JavaScript, and Google Sheets.

---

## Step-by-Step Guide

### 1. Create a Google Sheet

1. Open Google Sheets and create a new sheet.
2. Title the sheet **"Contact Form"**.
3. Ensure the column headers in the sheet match the `name` attributes of your HTML form fields (e.g., `name`, `email`, `message`).

### 2. Set Up Google Apps Script

1. In your Google Sheet, click on **Extensions > Apps Script**.
2. Rename the project to **"Contact Form"**.
3. Copy and paste the following script into the editor:

```javascript
var sheetName = "Sheet1";
var scriptProp = PropertiesService.getScriptProperties();

function intialSetup() {
  var activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  scriptProp.setProperty("key", activeSpreadsheet.getId());
}

function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000);

  try {
    var doc = SpreadsheetApp.openById(scriptProp.getProperty("key"));
    var sheet = doc.getSheetByName(sheetName);

    var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    var nextRow = sheet.getLastRow() + 1;

    var newRow = headers.map(function (header) {
      return header === "timestamp" ? new Date() : e.parameter[header];
    });

    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow]);

    return ContentService.createTextOutput(
      JSON.stringify({ result: "success", row: nextRow })
    ).setMimeType(ContentService.MimeType.JSON);
  } catch (e) {
    return ContentService.createTextOutput(
      JSON.stringify({ result: "error", error: e })
    ).setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}
```

## Now Create Your HTML Form

```HTML

<form name="submit-to-google-sheet">

  <input type="text" name="name" placeholder="Your Name" required>
  <input type="email" name="email" placeholder="Your Email" required>
  <textarea name="message" placeholder="Your Message" required></textarea>
  <button type="submit" id="btnSubmit">Save to Google Sheet</button>

</form>


```

## Add JavaScript to Handle Form Submission

```Javascript

const scriptURL = "yourURLHere";
const form = document.forms["submit-to-google-sheet"];

form.addEventListener("submit", (e) => {
  e.preventDefault();
  fetch(scriptURL, { method: "POST", body: new FormData(form) })
    .then((response) => {
      setTimeout(function () {
        msg.innerHTML = "";
      }, 2000);
      form.reset();
    })
    .catch((error) => console.error("Error!", error.message));
});

```

## Now Test Your Project
