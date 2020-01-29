# RFD-3566: Use Jurisdiction Specific Measures in Champ

> Finished Jan 03 2020

## Table of Contents

| Section | Name | Section | Name |
|---|---|---|---|
|I. | [Description](#description) | IV. | [Test Breaks and Fixes](#test-breaks-and-fixes) |
|II. | [Steps to Solution](#steps-to-solution) | V. | [Removed Files](#removed-files) |
|III. | [Components and Their Changes](#components-and-their-changes ) | | |

___

## Description

When a single application is viewed two api calls occur; one which retrieves application information, and another that returns a list of product categories and their respective products(referred to as measures). Due to the different jurisdiction laws in areas that Renew Financial services, certain products are restricted in different areas.

The purpose of this pull request is to custom tailor the returned list of measures to specific jurisdictions.

___

## Steps to Solution

### 1. Return All Measures Associated With An Application

- Initially an API call was made from an Angular component that used a `programId` and `sponsorId` taken from an application to make an API call to retrieve the measures available for an application. These measures were then dispatched to the redux store.

- This pull request will remove the API call and return the list of measures available for an application based off the program, sponsor, and jurisdiction from Panda. This data will be retrieved in the API call made when an application is used reducing the total number of API calls when viewing an application to 1.

### 2. Add Query String to API Call For An Application

- Initially when a user viewed an application, an Angular component made an API call that returned the details of an application without any measures.

- I added a query string to the URL that L. Canales used to include the measures in the response payload from Nemo.

### 3. Pass the Measures to the Redux Store

- Initially a saga was used to return the measures by giving the saga a program and sponsor id.

- I changed the saga so that it would only accepts an array of measures.

### 4. Update the fetchCategories Saga

- Initially the `fetchCategories` saga used an api call to gather measures and dispatch them to the redux store.

- This pull request will remove use of this API call as the measures are now returned with an application.

___


## Components and Their Changes

| Component | Path | Change |
| --- | --- | --- |
| application-details-breakdown | src/js/app/modules/rf-toolkit/directives/details/rf-application-details-breakdown.js : 38 | I replaced `const sponsor` with  `const productCategories` because the sponsor ID is no longer used to return measures. An array of measures will exist on an application, which I stored in `const productCategories` to be passed to the store. |
| application-details-lookup-factory | src/js/app/modules/rf-toolkit/services/application-details/application-details-lookup-factory.js : 132 | I added the query string `?include=pristine_product_categories` to the url used to return an application's details. The updated url now will return a payload with a `product_categories : [{measures}...]` key :  value. |
| application-details-breakdown | src/js/app/modules/rf-toolkit/directives/details/rf-application-details-breakdown.js : 45 | I used the `fetchCategories` saga and dispatched the measures to the store with `$ngRedux.dispatch(measuresActions.fetchCategories(productCategories));`. |
| fetch-categories | src/js/app/modules/rf-react/pipeline/application-details/measures/sagas/fetch-categories.js : 12 | I removed the import statement and use of the `fetchNonCustomCategories` API call and modified the argument given to the normalize function. |

___

## Test Breaks and Fixes

These changes broke a total of 28 test, with a majority of them being due to the updated url.

### 1. `actions should create an action to non-custom measures`

- PATH: _`src/js/app/modules/rf-react/pipeline/application-details/measures/__tests__/actions-test.js`_

- **Failed because** I updated the `fetchCategories` action to only take in an array of categories. The break occurs because the test expects a `programId` and a `sponsorId` to exist still.

- **Fixed by** Replacing the hard coded `programIdentifier` and `sponsorIdentifier` variables with a `categories` variable that holds an array with one object `{ name: "...", product_types: [{...}] }`.

### 2. `fetch measures saga fetchCategories fetches, normalizes, and receives measures`

- PATH: _`src/js/app/modules/rf-react/pipeline/application-details/measures/__tests__/sagas/fetch-categories-test.js_`

- **Failed because** the `fetchNonCustomCategories` API method is no longer used nor are the `programIdentifier` or `sponsorIdentifier`.

- **Fixed by**

  - Replacing the `programIdentifier` and `sponsorIdentifier` properties in the action object with a `categories` that uses the `nonCustomCategoriesResponse` mock data.
  - Removing the test for `fetchNonCustomCategories` since it is no longer used.

- **Updates** I changed the name by removing `fetches,` from the `it` statement since `fetchCategories` does not fetch categories anymore.

### 3. `applicatoinDetailsLookupFactory getApplicationDetails calls to endpoint for the application and program`

- PATH: _`src/js/app/modules/rf-toolkit/services/application-details/application-details-lookup-factory-spec.js`_

- **Failed because** the url that is used in the `whenGet` method does not include the update query string.

- **Fixed by** adding the query string to the url.

### 4. `applicationDetailsLookupFactory getApplicationDetails calls to endpoint for application details silently if silentLoad option is set`

- PATH: _`src/js/app/modules/rf-toolkit/services/application-details/application-details-lookup-factory-spec.js`_

- **Failed because** the url that is used in the `whenGet` method does not include the update query string.

- **Fixed by** adding the query string to the url.

### 5. `applicationDetailsLookupFactory getApplicationDetails error requesting application data sets error message when 404`

- PATH: _`src/js/app/modules/rf-toolkit/services/application-details/application-details-lookup-factory-spec.js`_

- **Failed because** the url that is used in the `whenGet` method does not include the update query string.

- **Fixed by** adding the query string to the url.

### 6. `applicationDetailsLookupFactory getApplicationDetails error requesting application data sets error message when 403`

- PATH: _`src/js/app/modules/rf-toolkit/services/application-details/application-details-lookup-factory-spec.js`_

- **Failed because** the url that is used in the `whenGet` method does not include the update query string.

- **Fixed by** adding the query string to the url.

### 7. `applicationDetailsLookupFactory getApplicationDetails error requesting application data sets error message when above 500`

- PATH: _`src/js/app/modules/rf-toolkit/services/application-details/application-details-lookup-factory-spec.js`_

- **Failed because** the url that is used in the `whenGet` method does not include the update query string.

- **Fixed by** adding the query string to the url.

### 8. `applicationDetailsLookupFactory getApplicationDetails error requesting application data sets error message when not 403 or 404 and below 500`

- PATH: _`src/js/app/modules/rf-toolkit/services/application-details/application-details-lookup-factory-spec.js`_

- **Failed because** the url that is used in the `whenGet` method does not include the update query string.

- **Fixed by** adding the query string to the url.

### 9. `applicationSyncFactory syncApplication syncs the application when it is not synced and it syncs after first attempt calls the callback and sets syncState to SYNCHRONIZED after the response is received FAILED`

- PATH: _'src/js/app/modules/rf-toolkit/services/application-details/application-sync-factory-spec.js'_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 10. `applicationSyncFactory syncApplication syncs the application when it is not synced and it syncs after first attempt waits for the http response if the timeout finishes`

- PATH: _'src/js/app/modules/rf-toolkit/services/application-details/application-sync-factory-spec.js'_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 11. `applicationSyncFactory syncApplication syncs the application when it is not synced and it syncs after first attempt waits for the timeout if http response is received first`

- PATH: _'src/js/app/modules/rf-toolkit/services/application-details/application-sync-factory-spec.js'_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 12. `applicationSyncFactory syncApplication syncs the application when it is not synced and it syncs after second attempt calls the callback and sets syncState to SYNCHRONIZED after the 2nd response is received`

- PATH: _'src/js/app/modules/rf-toolkit/services/application-details/application-sync-factory-spec.js'_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 13. `syncApplication syncs the application when it is not synced runs the sync status clock progresses the sync status to WARN after 2 seconds`

- PATH: _'src/js/app/modules/rf-toolkit/services/application-details/application-sync-factory-spec.js'_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 14. `applicationSyncFactory syncApplication syncs the application when it is not synced runs the sync status clock progresses the sync status to ERROR after 60 seconds`

- PATH: _'src/js/app/modules/rf-toolkit/services/application-details/application-sync-factory-spec.js'_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 15. `applicationSyncFactory syncApplication syncs the application when it is not synced runs the sync status clock stops the sync status clock after 60 seconds`

- PATH: _'src/js/app/modules/rf-toolkit/services/application-details/application-sync-factory-spec.js'_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 16. `applicationSyncFactory syncApplication syncs the application when it is not synced runs the sync status clock stops the sync status clock if the app becomes synchronized`

- PATH: _'src/js/app/modules/rf-toolkit/services/application-details/application-sync-factory-spec.js'_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 17. `applicationSyncFactory syncApplication syncs the application when tracking ga events fires an NLS sync event when a sync request occur`

- PATH: _'src/js/app/modules/rf-toolkit/services/application-details/application-sync-factory-spec.js'_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 18. `applicationSyncFactory syncApplication syncs the application when tracking ga events fires subsequent NLS sync events when future sync requests occur`

- PATH: _'src/js/app/modules/rf-toolkit/services/application-details/application-sync-factory-spec.js'_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 19. `applicationSyncFactory syncApplication syncs the application when tracking ga events fires subsequent NLS sync events when future sync requests occur`

- PATH: _'src/js/app/modules/rf-toolkit/services/application-details/application-sync-factory-spec.js'_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 20. `financingApplicationDetailsFactory saveProjectInfo Sends a PUT request to save improvement data`

- PATH: _`src/js/app/modules/rf-toolkit/services/application-details/financing-application-details-factory-spec.js`_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 21. `financingApplicationDetailsFactory saveMortgageOverrideInfo Sends a PUT request to save mortgage override info`

- PATH: _`src/js/app/modules/rf-toolkit/services/application-details/financing-application-details-factory-spec.js`_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 22. `financingApplicationDetailsFactory saveMortgageOverrideInfo dispatches the mortgageOverrideSubmitted action`

- PATH: _`src/js/app/modules/rf-toolkit/services/application-details/financing-application-details-factory-spec.js`_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 24. `financingApplicationDetailsFactory savePlan Sends a PUT request to save improvement data`

- PATH: _`src/js/app/modules/rf-toolkit/services/application-details/financing-application-details-factory-spec.js`_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `application.id` param in the url, and added the query string.

### 25. `financingApplicationDetailsFactory refreshApplication sends a GET request to refresh the application`

- PATH: _`src/js/app/modules/rf-toolkit/services/application-details/financing-application-details-factory-spec.js`_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `program` `application.Id` param in the url, and added the query string.

### 26. `financingApplicationDetailsFactory checkDocumentStatus sends a GET request to refresh the application`

- PATH: _`src/js/app/modules/rf-toolkit/services/application-details/financing-application-details-factory-spec.js`_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `program` `application.Id` param in the url, and added the query string.

### 27. `financingApplicationDetailsFactory checkDocumentStatus sets isFinancingDocsPhaseStarted to true if phase 2 is started and phase 1 is complete` 

- PATH: _`src/js/app/modules/rf-toolkit/services/application-details/financing-application-details-factory-spec.js`_

- **Failed because** the url used in the `expectGet` function called beforeEach test did not include the updated query string.

- **Fixed by** using string interpolation for the `program` `application.Id` param in the url, and added the query string.


> You get the idea...

___

## Removed Files

### [api.js](../assets/code/RFD-3566/api.js.md)

- Removed because the `fetchNonCustomCategories` method is no longer used to return measures.

### [api-test.js](../assets/code/RFD03566/api-test.js)

- Removed because the `api.js` file has been removed and these tests are no longer necessary.
