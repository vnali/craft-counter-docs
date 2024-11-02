# Counter plugin for Craft CMS
This Craft CMS plugin helps you to have the site's and page's counter and statistics widgets.

## License & Pricing
This commercial plugin is available through the [Craft plugin store](https://plugins.craftcms.com/developer/vnali).

## Requirement
Craft 5 and higher.

## Main features:
- Count site visits, visitors, and online users.
- Count page visits.
- Support GraphQL and Twig for fetching site and page statistics.
- Provide statistics widgets for user dashboards.

![widget1](https://github.com/user-attachments/assets/3fcaa805-ea4f-4213-be3f-c0c0db77c58a)


## Count sites and page visits and site visitors

### Count automatically
 - Go to the plugin settings page  `admin/counter/settings/general`.
 - Change the `Register counter automatically` setting to enable.
   - Enabling this setting will add a JavaScript script to the end of all site pages if the counter is enabled for the site, which sends a request to the backend on every visit.
   - If you want to support outdated browsers like IE11 in your site's front end, enable `Support outdated browsers` in plugin settings. Disabling this item makes requests faster and lighter on the front end.
- In the plugin settings, enable `Site Counter` for the sites on which you want to count visits.

### Count manually
  - If your site is statically cached, add this to the end of your frontend pages:  
  ```
  {% do view.registerAssetBundle("vnali\\counter\\assets\\CounterAsset") %}
  ```
  - If your site is not statically cached, you can call the count service directly in twig:
  ```
  {% set fullUrl = craft.app.request.getAbsoluteUrl() %}
  {% do craft.counter.count(fullUrl) %}
  ```

#### Attention:  
  - If you count visits by calling the count service directly in Twig, you can enable `Prevent access to counter via HTTP request` in plugin settings. In this case, requests to the backend counter via HTTP requests are ignored.
  - When `Register counter automatically` is enabled, we disable counting by calling the `count()` service in Twig to prevent counting a visit twice.
  - Please ensure that you set the site's timezone in the general settings and the `Start of the week` in the plugin settings before using this plugin. Changing these settings may affect some pre-calculated statistics (for example, new visitors, maximum online users, page visit statistics, etc.).

## Widgets
- `Visits`
  - It shows the number of visits in a given date range.
  - You can enable `show visitors` to display visitors' data on the chart; this data will only appear if the chart interval is set to daily or less.
  - There is an option called Ignore visits interval. By selecting this option, the total number of visits will include those that are ignored by the visit interval setting.
- `Visitors`
  - It shows the number of unique visitors in a given date range.
  - This plugin calculates visitors on a daily basis, so the number of visitors is limited to the previous hour, the current hour, today, yesterday, or a specific day.
  - The anonymized value of the visitor's IP is kept in the database by default. You can disable this option by setting the `anonymizeIp` option to false in the plugin config file.
- `Average Daily Visitors`
  - It shows the average number of unique visitors in a given date range.
- `Online Visitors`
  - It shows the number of online visitors based on the time you consider a visitor online after a visit.
  - You can specify the time you consider a visitor online via `Online threshold in seconds` in the widget setting.
    - This number can not be higher than the `Online threshold in seconds` set in the plugin settings.
  - When you specify the site as `all` in the widget setting, it counts a user who is online in site A and site B once.
 ![widget2](https://github.com/user-attachments/assets/caae18a4-a348-4a35-8f96-0e802719c302)

- `Max Online`
   - It shows the maximum number of online site visitors and when it happened in a given date range on a chart.
- `Top Pages`
  - It shows a list of top visited pages in a given date range for a site.
- `Trending Pages`
  - It shows a list of trending pages in a given date range and site.
  - When the growth of the pages is displayed as a percentage, pages that did not receive any visits in the previous date ranges—such as yesterday, the past week, and the past month—are not shown.
- `Declining Pages`
  - It displays a list of declining pages in terms of visits within a specified date range and site.
  - If a page does not have a visit in the current date range -today, this week, this month, this year- it does not show up in the result.
  - Obviously, this widget data can be more helpful when the current date range getting closer to the end -end of the day, end of the week, ....-
- `Not Visited Pages`
  - You can see a list of pages that are not visited in a specified date range but visited earlier.
- `Pages Visits Statistics`
  - By using this widget, you can view the latest statistics for every desired page of your site.
- `Next Visited Pages`
  - This widget allows you to analyze which pages users visit after a specific page within a defined number of seconds.
    - This widget requires a considerable amount of time to load, so it's best to apply it to shorter date ranges only on small to medium-sized websites.

### Attention:
- The user preference for the `Week Start Day` only affects the `visits`, `visitors`, `average daily visitors`, `online visitors`, and `max online` widgets for the `thisWeek` date range. It does not impact page-related widgets such as `Top Pages`, `Trending Pages`, `Declining Pages`, `Not Visited Pages`, `Pages Visits Statistics`, and `Next Visited Pages`. The calculations for this week and the previous week for page-related widgets are always based on the selected `start of the week` specified in the plugin settings.

## Fetching site and page counter statistics

You can fetch site and page statistics on your site via Twig or GraphQL.
  
#### Important about GraphQL
- To be able to run GraphQL queries, your token needs to have the appropriate permission for the queried item, site, and timeframe.
- Make sure that you only enable needed items for a token.
  - For example, if you want today's site visits counter for the front page of your primary site, you should limit the schema to querying `Site Visits`, `Today` date range, and `main` site. -because the used token and schema are exposed to the end users in this case-.
- We always pass a `t` argument with a unique value to prevent caching results.
- We can pass a `debugMessage` field in test environments to obtain useful information from GraphQL results. You should allow `querying the debug message` in the schema.


### Get site visits

#### Twig
` {% set visits = craft.counter.siteVisits(dateRange, startDate, endDate, siteId, ignoreVisitsInterval) %} `
- The supported date ranges are: `custom`, `thisHour`, `previousHour`, `today`, `yesterday`, `thisWeek`, `thisMonth`, `thisYear`, `past7Days`, `past30Days`, `past90Days`, `pastYear`.
- The `startDate` and `endDate` only should be set when the date range is set as `custom`.
- By passing `Ignore visits interval` as true, the output includes also visits that are ignored by the visit interval setting. The Default is `false`.
  - You can use this option to see how many visits happen in the `ignoreVisitsInterval` threshold.

##### examples
` {% set visits = craft.counter.siteVisits('today') %} `  
` {% set visits = craft.counter.siteVisits('today', null, null, '*') %} `  // today site's visits for all sites
` {% set visits = craft.counter.siteVisits('yesterday', null, null, '2') %} `  
` {% set visits = craft.counter.siteVisits('thisHour') %} `  
` {% set visits = craft.counter.siteVisits('past90Days', null, null, '5', true) %} `  // past 90 days site's visits for site id of 5 without ignoring visits interval  
` {% set visits = craft.counter.siteVisits('thisWeek') %} `  
` {% set visits = craft.counter.siteVisits('cutsom', '2024-01-10', '2024-09-01', '*') %} `  // 2024-01-10 to 2024-09-01 site's visits for all sites

#### GraphQL
```
// visits for today for all sites. we pass a `t` argument with a unique value to prevent caching results.
{
  counter(dateRange:"today", siteId: "*", ignoreVisitsInterval: true, t:timestamp) {
	visits
	debugMessage
  }
}

// Visits for a custom range for the site with an ID of 2.
{
  counter(dateRange:"custom", startDate:"2024-01-10", endDate:"2024-09-01", siteId: "2", t: timestamp) {
	visits
	debugMessage
  }
}
```


### Get site visitors

#### Twig
` {% set visitors = craft.counter.siteVisitors(dateRange, startDate, endDate, siteId) %} `
- supported date ranges are: `thisHour`, `previousHour`, `today`, `yesterday`, and `custom` (only for one day).
- startDate and endDate only should be set when the date range is set as `custom`. 

##### examples
` {% set visitors = craft.counter.siteVisitors('today') %} `  
` {% set visitors = craft.counter.siteVisitors('today', null, null, '*') %} `  
` {% set visitors = craft.counter.siteVisitors('yesterday', null, null, '2') %} `  
` {% set visitors = craft.counter.siteVisitors('thisHour') %} `  
` {% set visitors = craft.counter.siteVisitors('cutsom', '2024-01-10', '2024-01-10') %} ` // when custom is passed, only one day can be passed  

#### GraphQL
- You can pass `debugMessage` to get more information about errors.

 ```
//
{
  counter(dateRange:"today", siteId: "*", t: timestamp) {
	visitors
	debugMessage
  }
}

{
  counter(dateRange:"custom", startDate:"2024-01-10", endDate:"2024-01-10", siteId: "2", t: timestamp) {
	visitors
	debugMessage
  }
}

// Combine visits and visitors query:  
{
  counter(dateRange:"today", siteId: "*", t: timestamp) {
	visits
	visitors
	debugMessage
  }
}

// We can also combine visitors with visits since the custom date range is limited to one day (the visitors query does not support ranges longer than one day).
{
  counter(dateRange:"custom", startDate:"2024-01-10", endDate:"2024-01-10", siteId: "2", t: timestamp) {
	visits
	visitors
	debugMessage
  }
}

```

### Get the site's average daily visitors

#### GraphQL
- The supported date ranges are: `All`, `today`, `thisWeek`, `thisMonth`, `thisYear`, `past7Days`, `past30Days`, `past90Days`, `pastYear`, `custom`.

```
{
	counter(dateRange: "past7Days", siteId: "*") {
		averageVisitors,
    		debugMessage
	}
}
```

### Get online users

#### Twig

`{% set online = craft.counter.onlineVisitors($siteId, $onlineThreshold) %}`
- `$siteId`: if siteId is not passed, primary siteId is used. If `*` is passed, unique online visitors across all sites are returned.
- `$onlineThreshold`: Specifies the duration for which a user is considered online.

#### GraphQL

```
{
	counter(siteId: "*", onlineThreshold: 100, t: unique) {
		onlineVisitors
		debugMessage
	}
}
```


### Get maximum online users for a date range

#### Twig
`{% set maxOnline = craft.counter.maxOnline(dateRange, startDate, endDate, siteId) %}`.   
- The result is an array. The first item represents the number of maximum online users, and the second item indicates the date when the maximum number of online users occurred.
- The supported date ranges are: `custom`, `thisHour`, `previousHour`, `today`, `yesterday`, `thisWeek`, `thisMonth`, `thisYear`, `past7Days`, `past30Days`, `past90Days`, `pastYear`.

#### GraphQL
```
{
	counter(dateRange: "custom", startDate: "2024-08-25", endDate: "2024-08-30", siteId:"*", t: timestamp) {
		maxOnline
		maxOnlineDate
		debugMessage
	}
}
```


### Get page visit statistics for a page

#### Twig:
`{% set pageVisits = craft.counter.pageVisits($pageUrl, $siteId, $attributes) %}`.     
- `$pageUrl`: the page URL is required.  
- `$siteId`: if it is not passed, the primary siteId is used. If the site of the page is not important, pass `*`.  
- `$attributes`: an array of attributes which you want in return. The default is `['all', 'allIgnoreInterval', 'today', 'yesterday', 'thisWeek', 'previousWeek', 'thisMonth', 'previousMonth', 'thisYear', 'previousYear', 'lastVisit']`.  

#### GraphQL:
- If the `siteId` of the requested page is not passed, the primary site is sent.
  - You can pass `*` to get the first matched page without filtering the site.
- You can use `@dateConvert` directive with calendar, format, locale and timezone parameters to return the last visit in the intended format
```
{
	pageVisits(page: "https://xyz.test/page1", siteId: "*", t: uniqueParam) {
		all
		allIgnoreInterval
		thisYear
		thisMonth
		thisWeek
		today
		previousYear
		previousMonth
		previousWeek
		yesterday
		lastVisit@dateConvert(calendar: "gregorian", format: "yyyy-MM-dd HH:mm:ss EEEE", locale: "en_US", timezone: "UTC")
		debugMessage
	}
}
```


### Get top pages

#### Twig
` {% set topPages = craft.counter.topPages(dataRange, siteId, limit) %} `

  - The dataRange can be `all`, `allIgnoreInterval`, `today`, `thisWeek`, `thisMonth`, `thisYear`, `yesterday`.
  - If `siteId` is not passed, the primary site is used. If `*` is passed, the top pages for all sites will be returned.

#### GraphQL
  ```
  {
    topPages(dateRange: "all", siteId: "*", limit: 10, t: timestamp) {
	    page
	    visits
    }
  }
  ```


### Get trending pages

#### Twig  
`{% set trendingPages = craft.counter.trendingPages(dataRange, siteId, growthType, ignoreNewPages, limit) %}`

  - The `dataRange` can be `today`, `thisWeek`, `thisMonth`, or `thisYear`.
  - If `siteId` is not passed, the primary site is used. If `*` is passed, the top pages for all sites are passed.
  - The `growthType` can be passed as `count` or `percentage`. If `percentage` is selected the new pages are ignored. If `count` is selected, you can filter new pages via the `ignoreNewPages` argument.
  - If `ignoreNewPages` is set to true, pages without visits in the previous date range are not shown. The default value is false.

#### GraphQL:
```
{
  trendingPages(dateRange: "thisMonth", growthType: "count", ignoreNewPages: true, limit: 1) {
	page
	current
	previous
	growth
	debugMessage
  }
}
```


### More GraphQL examples

#### Get today's site visits and visitors for the primary site
- You should pass a new random value for the `t` argument like the current timestamp to prevent cache.
- You can skip passing siteId, by default primary siteId is used.
```
{
	counter(dateRange: "today", t: timestamp) {
		visits
		visitors
	}
}
```

#### Get site statistics in 2024-08-26 for site id 2
- By passing custom as dateRange, you can pass "2024-08-26" as the start date and end date.
- You can get current online visitors by passing `onlineVisitors` too.
- By passing the `debugMessage`, you can view a debug message if the result is ok or not.
- You can't pass visitors in this query because the selected `dateRange` is more than one day.
```
{
	counter(dateRange: "custom", startDate: "2024-08-26", endDate: "2024-08-26", siteId: "2", t: timestamp) {
		visits
		averageVisitors,
		onlineVisitors
		maxOnline
		maxOnlineDate
		debugMessage
	}
}
```

#### Get site statistics for yesterday for all sites and current online users in 30 seconds
- By passing `onlineVisitors` as a field, we get current online visitors with this query
- You can pass the `onlineThreshold` argument in seconds, the default value is the value set in plugin settings
```
{
	counter(dateRange: "yesterday", siteId: "*", onlineThreshold: 30, t: randomString) {
		visits
		visitors
		onlineVisitors
	}
}
```

## Config file
By copying the counter.php file to the config folder of your project, you can customize some plugin configurations.  
Config items are:  

- `anonymizeIp`: The default value is true, meaning the IP address is anonymized before processing and stored in the database.
- `ipInEvent`: The default value is false, indicating that the IP address is not included in the event.
- `anonymizedIpInEvent`: The default value is false, which means that the anonymized IP address is not included in the event.

## Events
 - EVENT_BEFORE_COUNT
   
   ```
	use vnali\counter\services\counterService;
	use vnali\counter\events\CountEvent;
	use yii\base\Event;
   	
	Event::on(
	    CounterService::class,
	    CounterService::EVENT_BEFORE_COUNT,
	    function(CountEvent $event) {
	        // to prevent counting
	        // $event->isValid = false;
	    }
	);
   ```
 - EVENT_AFTER_COUNT
   
   ```
	use vnali\counter\services\counterService;
	use vnali\counter\events\CountEvent;
	use yii\base\Event;
   	
	Event::on(
	    CounterService::class,
	    CounterService::EVENT_AFTER_COUNT,
	    function(CountEvent $event) {
		//	
	    }
	);
   ```
   
- Passed event items:
  
   ```
    public string $page; // The requested page, trimmed to 2048 characters or 3072 bytes
    public string $untrimmedPage; // The untrimmed requested page
    public int $siteId; // The siteId that page belong to
    public ?int $userId;  // The user who requested the page
    public ?string $ip;  // IP of the user. It is passed only if `ipInEvent` config is set to true.
    public ?string $anonymizedIp;  // The anonymized version of IP. It is passed only if `anonymizedIpInEvent` config is set to true.
    public string $hashedIp; // The hashed version of IP
    public ?string $userAgent; // The user agent who requested the page
    public DateTime $time; // The time when page is requested in UTC
   ```

## Permissions

Label | Permission | Description
--- | --- | ---
Manage plugin settings | *counter-manageSettings* | Can manage plugin settings
Access plugin widgets  | *counter-accessWidgets* | Can view counter widgets

## ToDo
- [x] Test on the Craft Cloud.

## Ideas
- [ ] Provide a caching mechanism for faster widget loads.
- [ ] Implement Ajax load for widgets for better user experience.
      

## FAQ

## Known issues

## Contact
Feel free to contact me by email at vnali.dev@gmail.com or direct message me via 'vnali' on [Craft CMS Discord](https://craftcms.com/discord) channel.

