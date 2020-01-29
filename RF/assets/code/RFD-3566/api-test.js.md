[Return to PR Doc](../../../pull_requests/jurisdiction_override.md)

## Original Path

- `src/js/app/modules/rf-react/pipeline/application-details/measures/__tests__/api-test.js`

___

```javascript
import { request } from 'rf-utils/api';
import { call } from 'redux-saga/effects';
import { fetchNonCustomCategories } from '../api';
import { nonCustomCategoriesResponse } from '../__mocks__/non-custom-categories-response';

describe('api', () => {
  describe('fetchNonCustomCategories', () => {
    it('requests for non-custom measures in the given program and sponsorship', () => {
      const generator = fetchNonCustomCategories('california_first', 'cscda');
      let next = generator.next();
      const url = '/programs/california_first/sponsors/cscda/measures?exclude_categories[]=custom';
      expect(next.value).toEqual(call(request, url));
      const response = {
        status: 200,
        json: nonCustomCategoriesResponse,
      };
      next = generator.next(response);
      expect(next.value).toEqual(response.json);
      expect(next.done).toEqual(true);
    });

    describe('when no json is returned', () => {
      let generator;
      let message;
      let response;
      beforeEach(() => {
        generator = fetchNonCustomCategories('california_first', 'cscda');
        generator.next();
        message = 'Ralph said the person who tries methods, ignoring principles, is sure to have trouble';
        response = {
          response: {
            body: {
              message,
            },
            status: 405,
            json: () => {},
          },
        };
      });
      it('throws an api error', () => {
        expect(() => { generator.next(response); }).toThrow();
      });

      it('throws an api error with correct error code', () => {
        try {
          generator.next(response);
          expect(true).toEqual(false);
        } catch (e) {
          expect(e.httpStatusCode).toEqual(response.response.status);
        }
      });
      it('throws an api error with correct error message', () => {
        try {
          generator.next(response);
          expect(true).toEqual(false);
        } catch (e) {
          expect(e.message).toEqual(message);
        }
      });
    });
  });
});
```