#  Google Sheet - Telegram Bot | RemindMe Bot

This project is a simple personal telegram bot connected to a google sheet.

## Video Demonstration
https://youtu.be/MFKLa266tDs


## Currently the bot has the following commands:

* /sheets - lists all the sheets in the connected workbook
* @[sheetname] text value -

* * Will create a new sheet if sheetname is not existing

## Technologies used for this project:

* Google Sheets
* Google App Script [1]
* Telegram

The telegram bot is connected to the google sheet using webhook.

![image explanation](https://i.stack.imgur.com/d93ud.png)

Webhooks are http messages that are sent as a result of an event.
In this particular example an event is a message sent from the telegram app
delivered to google servers via Telegram Bot Father.

Telegram sent an http post request (JSON) to the App Script.
The App Script will parse the JSON and will execute a function depending on the keyword.


[1] Google App Script is the scripting language Google uses and it is based on JavaScript and
is designed so the user can easily integrate with Google Workspace applications.

![sample script](https://www.benlcollins.com/wp-content/uploads/2019/08/GoogleAppsScriptMenu.jpg)

## How it works : Simple Rundown

After sending a message through the telegram thread, the sheet using the Google App Script will parse the message and execute a command depending on the keyword detected.

The commands are:

@sheetname space text - will add a text to the specified sheet
/sheets - will display all the sheets in the workbook

## Sample Code taken from the App

/* Constants definition */
var key = <insert_key_here>;
var url = "https://api.telegram.org/bot" + key;
var appUrl = "https://script.google.com/macros/s/<url>/exec"
var sheetId = <sheet_id>;
var today = Utilities.formatDate(new Date(), "GMT+8", "dd/MM/yyyy")
var eugId = <telegram_id>;



/* Establish webhook connection between Telegram and the app */
/* Need to run once */
function webhook(){
  var response = UrlFetchApp.fetch(url + "/setWebhook?url=" + appUrl);
}

/* To send message, Telegram needs the id of the recepient and the text to send */
function message(id, text){
  var response = UrlFetchApp.fetch(url + "/sendMessage?chat_id=" + id + "&text=" + encodeURIComponent(text));
}


function addNewItem(sheet, text){
  var sheetname =  SpreadsheetApp.openById(sheetId).getSheetByName(sheet);
  /* check if this sheet exist */
  if (sheetname){
    sheetname.appendRow([new Date(), text])
  } else {
    var currSheet = SpreadsheetApp.openById(sheetId).insertSheet(sheet);
    currSheet.appendRow([new Date(), text]);
  }
}

/*
function summarize(text){
  var textArray = text.slice(1).split(" ");
  var sheetname = textArray[1]
  var currSheet = SpreadsheetApp.openById(sheetId).getSheetByName(sheetname);
  if (currSheet){
    var summary = []
    var range = currSheet.getDataRange().getValues();
    if (range.length <= 10){
      length = range.length;
    } else {
      length = 10;
    }
    for (i = 0; i < length; i++){
      summary.push(range[i][1]);
    }
    var newMessage = summary.join("\n");

    message(id, newMessage);
  } else {
    message(id, "Sheet not found");
  }
}
*/


/* Will execute when the app receives a post request */
function doPost(e){
  var telegramMessage = JSON.parse(e.postData.contents);
  var id = telegramMessage.message.from.id;

  if (id != eugId) {
    newMessage = "Apologies, this is a personal bot and the access is for the creator only."
    message(id, newMessage);
  }

  var name = telegramMessage.message.from.first_name;
  var text = telegramMessage.message.text;


  if (text.startsWith("@")){
    var textArray = text.slice(1).split(" ");
    var sheet = textArray[0];
    textArray.shift();
    var textContent = textArray.join(" ");
    addNewItem(sheet, textContent);
    var newMessage = "Hi, " + name + ". Your item '" + textContent + "' has been added to the '" + sheet + "' sheet.";
    message(id, newMessage);
  }

  else if (text.startsWith("/sheets")){
    var sheetList = SpreadsheetApp.openById(sheetId).getSheets()
    var availableSheets = []
    for (i = 0; i < sheetList.length; i++){
      availableSheets.push(sheetList[i].getSheetName());
    }

    var newMessage = "Available sheets: " + availableSheets.join(", ");
    message(id, newMessage);
  }

  else {
    var newMessage = 'The command is not recognized. Available commands are: @[sheet] [text] - to append a text to a sheet, /sheets - to see all available sheets,'
    message(id, newMessage);
  }

}
