[Return to PR Doc](../../../pull_requests/jurisdiction_override.md)

## Original Path

- `src/js/app/modules/rf-react/pipeline/application-details/measures/api.js`

___

```Javascript
import { request, ApiError } from 'rf-utils/api';
import { call } from 'redux-saga/effects';

export function* fetchNonCustomCategories(programIdentifier, sponsorIdentifier) {
  const url = `/programs/${programIdentifier}/sponsors/${sponsorIdentifier}/measures?exclude_categories[]=custom`;
  const { json, response } = yield call(request, url);
  if (!json) {
    throw new ApiError(response.body.message, response.status);
  }

  return json;
}
```
