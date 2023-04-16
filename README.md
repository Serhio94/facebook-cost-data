// sheet url
const SPREADSHEET_URL = SpreadsheetApp.getActive().getUrl();
const ss = SpreadsheetApp.openByUrl(SPREADSHEET_URL);

// ad account ID
const AD_ACCOUNT_ID = '2925846997735988';

// user access token linked to a Facebook app
const TOKEN = 'EAAGZB03DKvswBAD46L0p74EQgResuQNuTnBML2RqUqNJ8UwROr0CBqQQIlkjrDTyQSkQ7KVbJOWZCd3mNEh5Ke0kqX2mi7la4NwZBI66aWcy04GR6Q7YQUdTSgpXGljJXHfaIFphCtGuUTqVzbZClSs5hjz43ZCqMqEmTC2wDlyXY4nUKIGac';

// variable from config tab
const LEVEL = 'account';
const START_DATE = '2023-01-01';
const END_DATE = '2023-12-31';
const FIELDS = 'spend';

// exchange rate from USD to BGN
const USD_TO_BGN = 1.74;

function requestReport() {

  // Builds the Facebook Ads Insights API URL
  const facebookUrl = `https://graph.facebook.com/v16.0/act_${AD_ACCOUNT_ID}/insights?level=${LEVEL}&fields=${FIELDS}&time_increment=1&time_range={'since':'${START_DATE}','until':'${END_DATE}'}&access_token=${TOKEN}`;
  Logger.log(facebookUrl)
  const encodedFacebookUrl = encodeURI(facebookUrl);
  const options = {
    'method': 'post'
  };

  // Fetches & parses the URL 
  const fetchRequest = UrlFetchApp.fetch(encodedFacebookUrl, options);
  const results = JSON.parse(fetchRequest.getContentText());

  // Caches the report run ID
  const reportId = results.report_run_id;
  const cache = CacheService.getScriptCache();
  const cached = cache.get('campaign-report-id');

  if (cached != null) {
    cache.put('campaign-report-id', [], 1);
    Utilities.sleep(1001);
    cache.put('campaign-report-id', reportId, 21600);
  } else {
    cache.put('campaign-report-id', reportId, 21600);
  };
  Logger.log(cache.get('campaign-report-id'));
  return reportId

}

// name of the sheet where the report will be
var TAB_NAME = 'FB cost'

function downloadReport() {


  var sheet = ss.getSheetByName(TAB_NAME);

  // Clears the sheet
  sheet.getRange('A:P').clearContent();

  // Gets the Facebook report run ID
  const reportId = requestReport();

  // sleep 30 seconds until report is ready
  Utilities.sleep(30000);
  // Fetches the report as a csv file
  const url = `https://www.facebook.com/ads/ads_insights/export_report?report_run_id=${reportId}&format=csv&access_token=${TOKEN}`;
  var fetchRequest = UrlFetchApp.fetch(url);
  const results = Utilities.parseCsv(fetchRequest);

  // Convert the cost data from USD to BGN
  for (var i = 1; i < results.length; i++) {
    results[i][5] = Number(results[i][5]) * USD_TO_BGN;
  }

  // Extract the csv file to the sheet
  sheet.get
