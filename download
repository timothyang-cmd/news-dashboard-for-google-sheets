/****************************************************
 * NEWS DASHBOARD FOR GOOGLE SHEETS
 *
 * Sources:
 * 1. NYT Top Stories
 * 2. NYT Most Popular
 * 3. Google News
 * 4. Yahoo News Japan RSS
 *
 * Output:
 * One sheet only: "News"
 *
 * Columns:
 * Source | Feed | Published | Title
 ****************************************************/


/****************************************************
 * CONFIGURATION
 ****************************************************/

const CONFIG = {
  DASHBOARD_SHEET_NAME: 'News',

  /******** NYT ********/
  NYT_API_KEY: getScriptProperty_('NYT_API_KEY'),
  NYT_SECTION: 'world',
  NYT_MAX_ITEMS: 3,

  /******** NYT MOST POPULAR ********/
  NYT_MOST_POPULAR_PERIOD: 1, // Use 1, 7, or 30
  NYT_MOST_POPULAR_MAX_ITEMS: 3,

  /******** GOOGLE NEWS ********/
  GOOGLE_NEWS_QUERIES: [
    'AI education',
    'Japan university education',
    'English education Japan'
  ],
  GOOGLE_NEWS_LANGUAGE: 'en',
  GOOGLE_NEWS_COUNTRY: 'JP',
  GOOGLE_NEWS_EDITION: 'JP:en',
  GOOGLE_NEWS_RESULTS_PER_QUERY: 3,

  /******** JAPANESE NEWS — YAHOO ONLY ********/
  JAPANESE_RSS_FEEDS: [
    {
      source: 'Yahoo News Japan',
      feedName: 'Yahoo Japan Topics - Domestic',
      url: 'https://news.yahoo.co.jp/rss/topics/domestic.xml',
      maxItems: 3,
      translateTitle: true
    },
    {
      source: 'Yahoo News Japan',
      feedName: 'Yahoo Japan Topics - World',
      url: 'https://news.yahoo.co.jp/rss/topics/world.xml',
      maxItems: 3,
      translateTitle: true
    },
    {
      source: 'Yahoo News Japan',
      feedName: 'Yahoo Japan Topics - IT / Science',
      url: 'https://news.yahoo.co.jp/rss/topics/it.xml',
      maxItems: 3,
      translateTitle: true
    }
  ],

  /******** GENERAL ********/
  MAX_MAIN_ROWS: 150
};


/****************************************************
 * MAIN FUNCTION
 * Run this function.
 ****************************************************/

function updateNewsDashboard() {
  const sheet = getOrCreateSheet(CONFIG.DASHBOARD_SHEET_NAME);

  sheet.getDataRange().breakApart();
  sheet.clear();

  const mainRows = [];

  mainRows.push([
    'Source',
    'Feed',
    'Published',
    'Title'
  ]);

  const nytRows = fetchNYTRows();
  const nytMostPopularRows = fetchNYTMostPopularRows();
  const googleNewsRows = fetchGoogleNewsRows();

  const combinedMainRows = mainRows
    .concat(nytRows)
    .concat(nytMostPopularRows)
    .concat(googleNewsRows)
    .slice(0, CONFIG.MAX_MAIN_ROWS + 1);

  sheet
    .getRange(1, 1, combinedMainRows.length, combinedMainRows[0].length)
    .setValues(combinedMainRows);

  const japaneseStartRow = combinedMainRows.length + 3;

  writeJapaneseNewsSection(sheet, japaneseStartRow);

  formatDashboard(sheet, japaneseStartRow);
}


/****************************************************
 * 1. NYT TOP STORIES
 ****************************************************/

function fetchNYTRows() {
  const rows = [];

  if (!CONFIG.NYT_API_KEY || CONFIG.NYT_API_KEY.includes('PASTE_')) {
    rows.push(makeErrorRow('NYT', CONFIG.NYT_SECTION, 'Missing NYT API key'));
    return rows;
  }

  const url = `https://api.nytimes.com/svc/topstories/v2/${CONFIG.NYT_SECTION}.json?api-key=${CONFIG.NYT_API_KEY}`;

  try {
    const response = UrlFetchApp.fetch(url, {
      method: 'get',
      muteHttpExceptions: true
    });

    const statusCode = response.getResponseCode();

    if (statusCode !== 200) {
      rows.push(makeErrorRow('NYT', CONFIG.NYT_SECTION, `NYT API error: ${statusCode}`));
      return rows;
    }

    const data = JSON.parse(response.getContentText());

    if (!data.results || data.results.length === 0) {
      rows.push(makeErrorRow('NYT', CONFIG.NYT_SECTION, 'No results returned'));
      return rows;
    }

    data.results.slice(0, CONFIG.NYT_MAX_ITEMS).forEach(article => {
      rows.push([
        'NYT',
        article.section || CONFIG.NYT_SECTION,
        formatPublishedDate(article.updated_date || article.published_date),
        makeHyperlink(article.url, article.title || 'Open article')
      ]);
    });

  } catch (error) {
    rows.push(makeErrorRow('NYT', CONFIG.NYT_SECTION, error.message));
  }

  return rows;
}


/****************************************************
 * 2. NYT MOST POPULAR
 ****************************************************/

function fetchNYTMostPopularRows() {
  const rows = [];

  if (!CONFIG.NYT_API_KEY || CONFIG.NYT_API_KEY.includes('PASTE_')) {
    rows.push(makeErrorRow('NYT Most Popular', 'Viewed', 'Missing NYT API key'));
    return rows;
  }

  const period = CONFIG.NYT_MOST_POPULAR_PERIOD || 1;
  const maxItems = CONFIG.NYT_MOST_POPULAR_MAX_ITEMS || 3;

  const url = `https://api.nytimes.com/svc/mostpopular/v2/viewed/${period}.json?api-key=${CONFIG.NYT_API_KEY}`;

  try {
    const response = UrlFetchApp.fetch(url, {
      method: 'get',
      muteHttpExceptions: true
    });

    const statusCode = response.getResponseCode();

    if (statusCode !== 200) {
      rows.push(makeErrorRow(
        'NYT Most Popular',
        `Viewed / ${period} day(s)`,
        `NYT Most Popular API error: ${statusCode}`
      ));
      return rows;
    }

    const data = JSON.parse(response.getContentText());

    if (!data.results || data.results.length === 0) {
      rows.push(makeErrorRow(
        'NYT Most Popular',
        `Viewed / ${period} day(s)`,
        'No results returned'
      ));
      return rows;
    }

    data.results.slice(0, maxItems).forEach(article => {
      rows.push([
        'NYT Most Popular',
        article.section || `Viewed / ${period} day(s)`,
        formatPublishedDate(article.published_date || article.updated),
        makeHyperlink(article.url, article.title || 'Open article')
      ]);
    });

  } catch (error) {
    rows.push(makeErrorRow('NYT Most Popular', `Viewed / ${period} day(s)`, error.message));
  }

  return rows;
}


/****************************************************
 * 3. GOOGLE NEWS RSS
 ****************************************************/

function fetchGoogleNewsRows() {
  const rows = [];

  CONFIG.GOOGLE_NEWS_QUERIES.forEach(query => {
    const encodedQuery = encodeURIComponent(query);

    const url =
      `https://news.google.com/rss/search?q=${encodedQuery}` +
      `&hl=${CONFIG.GOOGLE_NEWS_LANGUAGE}` +
      `&gl=${CONFIG.GOOGLE_NEWS_COUNTRY}` +
      `&ceid=${CONFIG.GOOGLE_NEWS_EDITION}`;

    try {
      const response = UrlFetchApp.fetch(url, {
        method: 'get',
        muteHttpExceptions: true
      });

      const statusCode = response.getResponseCode();

      if (statusCode !== 200) {
        rows.push(makeErrorRow('Google News', query, `Google News RSS error: ${statusCode}`));
        return;
      }

      const items = parseRssItems(response.getContentText());

      if (!items || items.length === 0) {
        rows.push(makeErrorRow('Google News', query, 'No results returned'));
        return;
      }

      items.slice(0, CONFIG.GOOGLE_NEWS_RESULTS_PER_QUERY).forEach(item => {
        rows.push([
          'Google News',
          query,
          formatPublishedDate(item.pubDate),
          makeHyperlink(item.link, item.title || 'Open article')
        ]);
      });

    } catch (error) {
      rows.push(makeErrorRow('Google News', query, error.message));
    }
  });

  return rows;
}


/****************************************************
 * 4. JAPANESE NEWS SECTION
 ****************************************************/

function writeJapaneseNewsSection(sheet, startRow) {
  sheet.getRange(startRow, 1, 1, 1).setValue('Japanese News Sources');

  const headerRow = [
    [
      'Source',
      'Feed',
      'Published',
      'Title'
    ]
  ];

  sheet.getRange(startRow + 1, 1, 1, headerRow[0].length).setValues(headerRow);

  const rows = fetchJapaneseNewsRows();

  if (rows.length === 0) {
    rows.push([
      'Japanese News',
      '',
      formatPublishedDate(new Date()),
      'No items found'
    ]);
  }

  sheet.getRange(startRow + 2, 1, rows.length, headerRow[0].length).setValues(rows);
}


function fetchJapaneseNewsRows() {
  const rows = [];

  CONFIG.JAPANESE_RSS_FEEDS.forEach(feed => {
    if (!feed.url || feed.url.includes('PASTE_')) {
      rows.push([
        feed.source,
        feed.feedName,
        formatPublishedDate(new Date()),
        'RSS URL missing'
      ]);
      return;
    }

    try {
      const response = UrlFetchApp.fetch(feed.url, {
        method: 'get',
        muteHttpExceptions: true
      });

      const statusCode = response.getResponseCode();

      if (statusCode !== 200) {
        rows.push([
          feed.source,
          feed.feedName,
          formatPublishedDate(new Date()),
          `RSS error: ${statusCode}`
        ]);
        return;
      }

      const items = parseRssItems(response.getContentText());

      if (!items || items.length === 0) {
        rows.push([
          feed.source,
          feed.feedName,
          formatPublishedDate(new Date()),
          'No items found'
        ]);
        return;
      }

      items.slice(0, feed.maxItems || 3).forEach(item => {
        let displayTitle = item.title || 'Open article';

        if (feed.translateTitle === true && item.title) {
          const englishTitle = translateJapaneseToEnglish(item.title);
          displayTitle = `${item.title} / ${englishTitle}`;
        }

        rows.push([
          feed.source,
          feed.feedName,
          formatPublishedDate(item.pubDate),
          makeHyperlink(item.link, displayTitle)
        ]);
      });

    } catch (error) {
      rows.push([
        feed.source,
        feed.feedName,
        formatPublishedDate(new Date()),
        `RSS parsing error: ${error.message}`
      ]);
    }
  });

  return rows;
}


/****************************************************
 * RSS PARSER
 ****************************************************/

function parseRssItems(xmlText) {
  const document = XmlService.parse(xmlText);
  const root = document.getRootElement();

  let channel = root.getChild('channel');
  let items = [];

  // RSS 2.0
  if (channel) {
    items = channel.getChildren('item');

    return items.map(item => {
      return {
        title: item.getChildText('title') || '',
        link: item.getChildText('link') || '',
        pubDate: item.getChildText('pubDate') || item.getChildText('date') || ''
      };
    });
  }

  // RSS 1.0 / RDF fallback
  items = root.getChildren('item');

  return items.map(item => {
    return {
      title: item.getChildText('title') || '',
      link: item.getChildText('link') || '',
      pubDate: item.getChildText('date') || ''
    };
  });
}



/**
 * Reads a secret from Apps Script Project Settings > Script Properties.
 * Do not hard-code API keys in this file.
 */
function getScriptProperty_(key) {
  return PropertiesService.getScriptProperties().getProperty(key) || '';
}

/**
 * Optional helper: run once to save your NYT API key in Script Properties.
 * Replace PASTE_YOUR_NYT_API_KEY_HERE, run this function, then remove the key again.
 */
function setNYTApiKeyOnce() {
  PropertiesService.getScriptProperties().setProperty(
    'NYT_API_KEY',
    'PASTE_YOUR_NYT_API_KEY_HERE'
  );
}

/****************************************************
 * HELPER FUNCTIONS
 ****************************************************/

function getOrCreateSheet(sheetName) {
  const spreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  let sheet = spreadsheet.getSheetByName(sheetName);

  if (!sheet) {
    sheet = spreadsheet.insertSheet(sheetName);
  }

  return sheet;
}


function formatDashboard(sheet, japaneseStartRow) {
  const lastRow = sheet.getLastRow();

  if (lastRow < 1) return;

  // Main header
  sheet.getRange(1, 1, 1, 4)
    .setFontWeight('bold')
    .setBackground('#d9ead3');

  // Japanese section title
  sheet.getRange(japaneseStartRow, 1, 1, 4)
    .merge()
    .setFontWeight('bold')
    .setBackground('#fff2cc')
    .setFontSize(12);

  // Japanese section header
  sheet.getRange(japaneseStartRow + 1, 1, 1, 4)
    .setFontWeight('bold')
    .setBackground('#fce5cd');

  sheet.setFrozenRows(1);

  const fullRange = sheet.getRange(1, 1, lastRow, 4);
  fullRange.setWrap(true);
  fullRange.setVerticalAlignment('top');

  sheet.setColumnWidth(1, 150); // Source
  sheet.setColumnWidth(2, 240); // Feed
  sheet.setColumnWidth(3, 140); // Published
  sheet.setColumnWidth(4, 720); // Title

  sheet.autoResizeRows(1, lastRow);
}


function cleanHtml(text) {
  if (!text) return '';

  return text
    .replace(/<[^>]*>/g, '')
    .replace(/&amp;/g, '&')
    .replace(/&quot;/g, '"')
    .replace(/&#39;/g, "'")
    .replace(/&apos;/g, "'")
    .replace(/&lt;/g, '<')
    .replace(/&gt;/g, '>')
    .replace(/&nbsp;/g, ' ')
    .trim();
}


function formatPublishedDate(value) {
  if (!value) return '';

  const date = new Date(value);

  if (isNaN(date.getTime())) {
    return value;
  }

  return Utilities.formatDate(
    date,
    Session.getScriptTimeZone(),
    'MMMM d yy'
  );
}


function makeHyperlink(url, label) {
  if (!url) return label || '';

  const safeUrl = String(url).replace(/"/g, '""');
  const safeLabel = String(label || url).replace(/"/g, '""');

  return `=HYPERLINK("${safeUrl}", "${safeLabel}")`;
}


function translateJapaneseToEnglish(text) {
  if (!text) return '';

  try {
    return LanguageApp.translate(text, 'ja', 'en');
  } catch (error) {
    return 'Translation unavailable';
  }
}


function makeErrorRow(source, feed, message) {
  return [
    source,
    feed,
    formatPublishedDate(new Date()),
    'ERROR: ' + message
  ];
}