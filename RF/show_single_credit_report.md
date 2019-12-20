# Champ Pull Request RFD-3695

> _Dec 2019_

## Table of Contents

| Section | Name | Section | Name |
|---|---|---|---|
| I. | [Description](#description) | V. | [Tests](#tests) |
| II. | [Components Involved](#components-involved) | VI. | [Important Files](#important-files) |
| III. | [Data Flow](#data-flow) | VII. | [Altered Files](#alterted-files) |
| IV. | [Plan](#plan) | VIII. | [Code Snippets](#code-snippets) |

___

## Description

Champ currently displays multiple notifications about credit reports, but each notification displays the same information. This is redundant and should be fixed such that only one credit report issue is displayed. Additionally, when a user views the pipeline, any applications that have issues display a count of the issues:

![pipeline](../assets/img/pipeline.png)

This text should not contain a numerical value.
___

## Components Involved

- `application-container-saga.js`

  - PATH: _src/js/app/modules/rf-react/pipeline/applications/sagas/applications-container-saga.js_
  - When viewing the pipeline, the state object contains an empty array for holding application issues (however there is a count of issues available). It is only by clicking the collapsible menu that this array is filled with issue objects. These objects originate from the  `fetchApplicationDetails` action being dispensed to the `fetchApplicationDetails` method in `applications-container-saga.js`.

___

## Data Flow

| Step | Description | Component |  Path | Action | Children Until Next Step |
|---|---|---|---|---|---|
| 1. | Get applications from Panda | `applications-container.jsx` | src/js/app/modules/rf-react/pipeline/applications/components/applications-container.jsx | An action is dispatched to return all applications. Applications are then accessible through giving the state value to a selector. [Click here to view the selector code](#selector-for-applications). This data is then passed as props to children. | `<ApplicationsTableContainer />` **=>** `<ApplicationRow />` **=>** `<CurrentSteps />` **=>**  `<CurrentStep />`| 
| 2. | Get the issues pertaining to a particular application by fetching an application via id and programId. Pass this along as a prop called `stipulations`. | `CurrentStep.jsx` | src/js/app/modules/rf-react/pipeline/applications/components/current-step.jsx | At this point in the pipeline if there are any issues found with the application, a collapsible menu with a count of the issues is available. If a user clicks on the collapsible menu, then the application dispatches an action to a saga to fetch application details. [Click here to see the code](#fetching-application-details). |  **=>** `<ApplicationApprovalStep />` **=>** `<ApplicationApprovalStepDetails />` **=>** `<StepDetails />` **=>** `<CollapseBody />`  No further children |
| 3. | Create an object with key value pairs of `appliactionID` : `{...application}`. Each pair is passed to the issue component. | `Issues.jsx` |  | |

___

## Plan

### To Fix Details in the Pipeline

**Problem**: When viewing the Pipeline there are two problems
  
  1. When the pipeline is loaded, PANDA sends a large amount of data to CHAMP. Included in this payload is an `applicationById` property that holds some, but not all information about applications. Each application has a property called `stipulations`. This property holds information pertaining to issues within an application. PANDA sends a count of the issues with an application, but not the issues themselves. This creates an issue when there are duplicate stipulations because the `issueCount` from PANDA does not filter for duplicates, so the count from PANDA might be incorrect.
      - `issueCount : 4` rather than `issueCount : 3`

**Solution**: Filter duplicates from the `stipulations.application_approval` array generated in the `applications-container-saga`. [Click to see code](#saga-filter-duplicate-issues).

___

## Tests

In order to ensure that the solution to this ticket is solved I will add a test case to the  `updateApplication` method in `applications-container-saga-test.js`.

To view the test case [Click Here](#saga-filter-duplicate-issues-test)

### Explanation



___

### Important Files

- `Issues.jsx`: used to display stipulations per applications.

- The pipeline uses a selector to get filtered apps.

- `application-approval-step-details.jsx` uses a count of all the issues, however the the count number does not contain information about this issues.  
  - `issueCount` : `1`
  - `issues` : `[]`

- `selectors.js` for the application selectors has three selectors that target issues:
      1. `unresolvedIssueSelector(issues)`
      2. `unresolvedIssueWithPreNTPSelector(issues)`
      3. `resolvedIssueSelector(issues)`

___

### Code Snippets

#### Selector for Applications

> PATH: _src/js/app/modules/rf-react/pipeline/applications/selectors.js_

```javascript
function getFilteredApps(state) {
  const byId = byIdSelector(state);
  const currentSet = searchSelector(state).currentSet;
  return _.map(currentSet, (appId) => {
    const app = byId[appId];
    return fp.set('project.applicants', getApplicants(app), app);
  });
}
```

___

#### Fetching Application Details

> PATH: __src/js/app/modules/rf-react/pipeline/applications/components/current-steps.jsx

```javascript
function mapDispatchToProps(dispatch) {
  return bindActionCreators({
    fetchApplicationDetails: fetchApplicationDetailsAction,
  }, dispatch);
}
```

> PATH: _src/js/app/modules/rf-react/pipeline/applications/components/current-step.jsx_

```javascript
function getStepDetailsProps(application, fetchApplicationDetails) {
  return {
    detailsCollapseProps: {
      apn: _.get(application, 'apn'),
      county: _.get(application, 'project.address.county'),
      hasLoaded: applicationsSelector.hasAppDetailsLoaded(application),
      isLoading: applicationsSelector.isAppDetailsLoading(application),
      fetchApplicationDetails: () => fetchApplicationDetails(application.id, application.program.id),
    },
    loadingIssuesProps: {
      hasLoadingError: applicationsSelector.doesAppHaveLoadingError(application),
      hasConnectionIssues: applicationsSelector.hasConnectionIssues(application),
    },
  };
}

// ...
// ...

 switch (step.key.toLowerCase()) {
    case 'application_approval':
      return (
        <ApplicationApprovalStep
          href={href}
          header={step.header}
          stipulations={_.get(application, 'stipulations.applicationApproval', {})}
          stepDetails={getStepDetailsProps(application, fetchApplicationDetails)}
        />);
```

___

#### Saga Filter Duplicate Issues

>PATH: _src/js/app/modules/rf-react/pipeline/applications/sagas/applications-container-saga.js_

```javascript
try {
    yield put(actions.setApplicationDetailsAsLoading(appId, programId));
    const application = yield call(api.fetchApplicationDetails, appId);

   // Solution to remove duplicates
    application.decisioning.stipulations.application_approval = _.filter(application.decisioning.stipulations.application_approval, (issue, index, self) =>
    index === self.findIndex(i => (i.name === issue.name))
    );

    if (callingFromGenerateDocs) {
      _.set(application, 'post_approval', true);
    }
    yield call(updateApplication, { data: { application } });
  }
```
___

#### Saga Filter Duplicate Issues Test

```javascript
it('it filters duplicate stipulations', () => {
      data.application = applicationWithIssues;
      const appStipulationIssues = applicationWithIssues.decisioning.stipulations.application_approval;
      generator = saga.updateApplication({ data });
      const app = applicationMapper.toApplication(applicationWithIssues, urlParams);
      next = generator.next(app);
      const issuesArray = next.value.PUT.action.data.stipulations.applicationApproval.issues;
      expect(issuesArray.length).toEqual(appStipulationIssues.length - 1);
    });
  });
```

[Return to code section](#tests)

___

#### Update Application Saga Pass Object

```JSON
LOG: 'NEXT : ', '{
  "value": {
    "@@redux-saga/IO": true,
    "CALL": {
      "context": null,
      "args": [
        {
          "data": {
            "application": {
              "id": 12345
            }
          }
        }
      ]
    }
  },
  "done": false
}'
```

#### Update Application Saga Fail Object

```JSON
LOG: 'NEXT : ', '{
  "value": {
    "@@redux-saga/IO": true,
    "PUT": {
      "channel": null,
      "action": {
        "type": "applications/FETCH_APPLICATION_DETAILS_ERROR",
        "data": {
          "appId": 12345,
          "error": "error"
        }
      }
    }
  },
  "done": false
}'
```

___
