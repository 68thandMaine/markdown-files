# Use API to Get County Taxrole Cutoff Information

## I. Introduction

### - Objective

Design a more efficient way for Champ to use data from the Repayment Estimator.

### - Problem

Currently Champ references an internal file `rf-pace-program-values` to county specific taxroll cutoff dates. This information is generally updated in the Repayment Estimator app using yml files. There is no need for Champ to have an internal file that contains the same information that the Repayment Estimator uses. Including this file increases tech debt, and can be a source of confusion if changes to county taxrolls are made, but only updated in the Repayment Estimator.

### - Solution

The easiest way to address this issue would be to remove Champ's reliance on the `rf-pace-program-values` file, and instead use an API to return these values.

___

## II. Required Changes

To impalement this functionality the following applications will need to be updated:

- [Champ](#updates-to-champ)
- [Nemo](#updates-to-nemo)
- [Repayment Estimator](#updates-to-the-repayment-estimator)

## Updates to Champ

Champ will need to have a way to retreive the most recent taxroll values from the Repayment estimator. This will involve the creation of a new API call, so several updates will need to be made to support this functionality.

> Dir paths in bold are newly created folders and files

| Update | File | Reason |
|---|---|---|
| [Add action types](#add-action-types) | src/js/app/modules/rf-react/calculator/inputs/constants/action-types.js | To update the slice of state holding county information Champ will need to dispatch several new actions to the calculator Reducer. |
| [Add new actions](#add-new-actions) | src/js/app/modules/rf-react/calculator/inputs/actions.js | To update the state, new actions must be sent to the calculator reducer. |
| [Update the Calculator Reducer](#update-the-calculator-reducer) | src/js/app/modules/rf-react/calculator/inputs/reducer.js | The reducer will now need to create new slices of state for handling the response of the API request. |
| [Create a Saga](#create-a-saga) | src/js/app/modules/rf-react/calculator/inputs/**sagas/fetch-county-taxroll-data.js** | This saga will handle fetching the values stored in the Repayment Estimator and updating the reducer |
| [Add an API method](#add-an-api-method) | src/js/app/modules/rf-react/calculator/inputs/**api/api.js** | This file will contain the API method to request county data based off of a program ID. |
| [Update the calculator directive](#update-the-calculator-directive) | src/js/app/modules/rf-toolkit/pace-calculator/directives/rf-calculator.js | The directive assembles all the information used in the calculator. This includes creating state objects. Code already exists here that   |

### Add Action Types

The following new action types will be created:

```javascript
    const FETCH_COUNTY_TAXROLL_DATA = 'calculator/inputs/FETCH_COUNTY_TAXROLL_DATA';
    const RECEIVE_COUNTY_TAXROLL_DATA = 'calculator/inputs/RECEIVE_COUNTY_TAXROLL_DATA';
    const RECEIVE_COUNTY_TAXROLL_DATA_ERROR = 'calculator/inputs/RECEIVE_COUNTY_TAXROLL_DATA_ERROR';
```

### Add New Actions

To support the three action types that will need to be created for this functionality, three new actions will be created as well:

```javascript

function fetchCountyTaxrollData(programIdentifier) {
    return {
        type: FETCH_COUNTY_TAXROLL_DATA,
        programIdentifier,
    }
}

function receiveCountyTaxrollData(response) {
    return {
        type: RECEIVE_COUNTY_TAXROLL_DATA,
        response,
    };
}

function receiveCountyTaxrollDataError(message, httpStatusCode) {
    return {
        type: RECEIVE_COUNTY_TAXROLL_DATA_ERROR,
        error: {
            message,
            httpStatusCode,
        },
    };
}

```

The new actions that are created are based off of the pattern that was originally used to return measures prior to eliminating the API call responsible for returning measures. Fetching the categories will set a state value to `isLoading: true` and the other two are dependent on the response emitted from the Saga.

### Update the Calculator Reducer

To update the calculator reducer to work with the new API call, a reducer function for handling fetching and receiving program counties will created and a reducer for handling API errors will be created. Then, these three new reducers will be exported with the current reducer function with the `combineReducer` method.

```javascript
import { combinReducers } from 'redux'
// import { actionTypes } from './actionTypes';

export function isLoading(state = {}, action = {}) {
    switch(action.type) {
        case FETCH_COUNTY_TAXROLL_DATA:
            return true;
        case RECEIVE_COUNTY_TAXROLL_DATA:
        case RECEIVE_COUNTY_TAXROLL_DATA_ERROR:
            return false;
        default:
            return state;
    }
}

export function error(state = {}, action = {}) {
    switch(action.type) {
        case RECEIVE_COUNTY_TAXROLL_DATA_ERROR:
            return {
                ...state,
                ...action.error,
            };
        case FETCH_COUNTY_TAXROLL_DATA:
            return {};
        default:
            return state;
    }
}

export function countyTaxrollValues(state = {}, action = {}) {
    switch(action.type) {
        case RECEIVE_COUNTY_TAXROLL_DATA:
            return action.rawResponse // <- could be named something else
        case FETCH_COUNTY_TAXROLL_DATA:
            return [];
        default:
            return state;
    }
}

function reducer(action){...} // <- existing reducer

export default combineReducers({
    isLoading,
    error,
    countyTaxrollValues,
    reducer,
})
```

Creating three new reducers is necessary because each will handles a small slice of state that is dependent on the API call. This follows the pattern used in the measures reducer where receiving categories, fetching categories, and handling any API errors is handled by several small reducers all exported with a `combineReducer` method.

### Create a Saga

To update state when the API is called a generator function will should be written and a helper function that watches FETCH_COUNTY_TAXROLL_DATA will fire the generator each time the action is triggered.

```javascript

function* fetchTaxrollCutoffs(action) {
    try {
        const response = yeild (call fetchProgramDefinitions, action.program);
        yeild put(actions.receiveCountyData(response);
    } catch (e) {
        yeild put(actions.receiveCountyDataError(e.message, e.httpStatusCode));
    } else {
        throw e;
    }
}

function* watchFetchTaxrollCutoffs() {
    yeild call(takeLatest, FETCH_COUNTY_TAXROLL_DATA, fetchTaxrollCutoffs)
}

export {
    fetchTaxRollCutoffs,
    watchFetchTaxrollCutoffs
}

```

The `watchFetchTaxrollCutoffs` "listens" for actions with a type property of `FETCH_COUNTY_TAXROLL_DATA` and fires the `fetchTaxRollCutoffs` method capturing the response of the last request.

### Add an API Method

Things to consider:

- What is the endpoint I want to hit?
- Add the following params to specify which counties to return?
  - program
  - programId
  - sponsor
  - sponsorId
- What use do I have for the {request, ApiError} imports?
- Should I follow the API request in calculator/output or in the measures?

```javascript
import { request, ApiError } from 'rf-utils/api';
import { call } from 'redux-saga/effects';

export function* fetchProgramCountyData(requestBody) {
    const url = 'nemo endpoint for the repayment estimator';
    const params = {
        method: 'GET'
        body: JSON.stringify(requestBody);
    }
    const { json, response} = yeild call(request, url, params);
    if (!json) {
        const errorJson = yeild call([response, response.json]);
        throw new ApiError(errorJson.message, errorJson.status, errorJson.requestCode);
    }
    return json;
}

```

### Update the Calculator Directive

The Angular directive rfCalculator creates the calculator data. In keeping with the logic of fetching measures, the directive file should dispatch the action for fetching the county taxroll cutoff dates. Thanks to the generator helper function, the saga will be called when directive code is hit.

To be more specific we could add in sponsor information here too.

```javascript
    scope.program = $stateParams.programId // <- exists in code base already

    $ngRedux.dispatch(calculatorActions.fetchCountyTaxrollData(scope.program)
```

___

## Updates to Nemo

Nemo currently does not support a GET request to the Repayment estimator - it only has two POST requests. This mean that Nemo will need to be changed so that it can make handle GET requests to the repayment estimator.

| Update | File | Reason |
|---|---|---|
| [Add GET route](#add-a-get-route) | config/routes.rb | This will setup route handling from Champ to Nemo to eventually the Repayment Estimator|
| [Create controller method](#create-controller-method) |controllers/repaymnt_estimatr_controller.rb | A new controller method will need to be written to direct Champ's API call to the appropriate Repayment Estimator endpoint. |
| [Add new endpoint to the config file](#update-configuration-file) | lib/services/repaymnt_estimatr/configuration.rb | To keep with the established pattern I will add the repayment estimator API endpoint to the config file. |
| [Create the Request to Repayment Estimator](#create-the-request) | lib/services/repaymnt_estimatr/batch_request.rb | THis request will return the counties with tax data. Or perhaps more. There are things to consider. |

### Add a GET route

Nemo's routes are grouped by type/concern of action in one file. This API call's purpose is to return a list of information that at a minimum must match what Champ currently uses.

There are there are two values that we are concerned with: a program and a sponsor. Currently there are only two programs available - each with one sponsor. So for now just the program needs to be dynamic:

```ruby
    match: "/programs/:program_identifier/sponsor/:sponsor_identifier/county_data"   to: "repaymnt_estimatr#get_county_tax_data", methods: [:get]
```

This code maps to a route in `repaymnt_estimatr_controller.rb` file, and takes two arguments: 

- programID
- sponsorID

### Create Controller Method

The new controller method will look very simple, but the code has many moving parts.

```ruby

def get_county_tax_data
    wrap_request do
        requester.batch_county_tax_response
    end
end

```

This will use the `bactch_request` method to construct the call and return the JSON response from the Repayment Estimator.

### Update configuration file

`configuration.rb` has funcitons that return the paths to specific endpoints to act as a single source of truth. I will add the repayment estimator endpoints to this file.

```ruby
    class Configuration
        
        # ....
        
        SPONSOR_TAX_DATA_ENDPOINT = '/... tbd '
        
        # ...
        
        def from_sponsor_tax_data_endpoint
            SPONSOR_TAX_DATA_ENDPOINT
        end
        
        # ...
    end
```

### Create the Request

Inside the `get_county_tax_data` method a method on the requestor class is called. This method contains the API method that will communicate with the Repayment Estimator.

```ruby

def batch_county_tax_response
    batch_request(
            :endpoint: => Configuration.from_sponsor_tax_data_endpoint,
            :body_proc => proc { | programId, sponsorId |
            params_object.program_hash(program: programId).to_json
            params_object.sponsor_hash(sponsor: sponsorId).to_json }
    )
end

# Could use the current code or create a new one. I will copy
# the existing code pattern in this example.

def batch_request(endpoint:, body_proc:)
    # can I  have multiple params in a do? If not, my data shape could be
    # an issue since not all keys are the same type.
        params_object.terms.each_with_object({}) do |term, responses|
          response = connection.get do |req|
            req.url endpoint
            # Again...multiple params? Is it necessary at all?
            req.body = body_proc.call(term)
            req.headers = Configuration.headers
          end
          if response.success?
            responses[term] = JSON.parse(response.body)
          end
        end
      end


```

I will need to do a little bit more researching into ruby routing, but this is the general idea for what should happen.

___
___

## Updates to the Repayment Estimator

THe Repayment Estimator App will require a bit of hacking away to assemble the information that we want. Additionally, there could be other data that should be returned from the Repayment Estimator which might need to be built into the response. This section will address returning county tax roll cutoff data only.

The information tht is being requested is stored in two different places. Champ uses a data shape like so:

```javascript
{
    "county_mame": string,
    "tax_roll_cutoff" : array_of_dates
}
```

This information is stored in the Repayment Estimator in such a way that the `county_name` property is a file name, and the `tax_roll_cutoff` is information that may or may not exist within a given file. To meet the needs of this feature, we will need the following information from an API call:

- program
- sponsor
- county names
- base tax roll cutoff values
- county specific tax roll cutoffs

With this information we can assemble the appropriate information for a response that can be used in Champ.

| Update | File | Reason |
|---|---|---|
| [Add New Route](#add-new-route) | repayment_estimator/config/routes.rb | The Repayment Estimator will need a new route to return the required data. |
| [Add New Controller Method](#add-new-controller-method) | repayment_estimator/app/controllers/repayment_estimator/program_definitions_controller.rb | Since this is a new request and no existing routes return the data we need, we will need to create a new controller method. |
| [Add New Definition](#add-new-definition) | |
| [Add New Method for Gathering County Names and Tax Dates](#add-method-for-taxroll-cutoffs-by-county) | |

### Add New Route

The new route that is added to the Repayment Estimator should be grouped with `program_definitions` because the route Champ uses in the calculator goes through programs. Since a user can select a program in Champ's calculator menu, it makes sense to start by having 1 API call per program to return tax values.

#### Possible new routes

```ruby
get "program_definitions/:program/",    to: "program_definitions#program_county_tax_roll"
```

> _Note there is already a route to `program_definitions/:program` but currently it only returns sponsor names_

### Add New Controller Method

```ruby

def taxroll_dates
    @program = params["program"]
    # @sponsor = params["sponsor"]
    @version = params["version"]

    @data = JSON.pretty_generate(program_definition)

```

### Add New Definition

#### Step 1 Update `hierarchy_of_definitions`

```ruby
 def hierarchy_of_definitions
      # NOTE: Order dependent from most generic (first) to most specific (last)
      [
        version_definition,
        program_base_definition,
        county_taxroll_values, # new definition
        sponsor_base_definition,
        county_definition
      ].compact
    end
```

#### Step 2 Add new helper method

```ruby
  def county_taxroll_values
    return unless program || sponsor

    load_yml!()
```

### Add Method for Taxroll Cutoff by County

- This will store an instance variable that holds the payload I want to send to Champ. This method is where the meat of the API call will be assembled. Most of the existing methods all use some sort of path to be fed to a helper method that reads YML files.

```ruby
    def taxroll_
```
___
# Headings: 1
## Headings: 2
### Headings: 3
#### Headings: 4
##### Headings: 5
###### Headings: 6