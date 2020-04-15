# Docusign Authentication Status in Panda

Docusign sends connect payloads with information about the Envelope (collection of documents that customers need to sign) each time an event of interest occurs. Customers are prompted to sign their documents in Docusign via email or through the customer portal. Docusign is a protected service, and at RF we use the Lexis Nexus Knowledge Based Authentication offred through Docusign. Customers are required to answer two pages of identity based questions before being allowed to sign documents. If a customer fails either one of these questions twice, then they are kicked out of Docusign, and call center reps have to resend the Docusign email from Panda.

**So what if call center reps want to know why a user is failing Docusign Login?**

That's where [RFD-4090](https://jira.renewfund.com/browse/RFD-4090) comes into play. This ticket asks for the Docusign login status to be displayed in the Documents view of Panda.

## Notable Behaviors

| Behavior | Payload |
| --- | --- |
| If a user fails the address / DOB / SSN authentication | ![](../assets/images/failed_lookup.png) |
| If a user passes the address / DOB / SSN authentication but fails the questions | ![](../assets/images/failed_questions.png) |
| If a user passes all authentication, or has not signed into Docusign | The `<RecipientAuthenticationStatus>` block is not returned.|

___

## Approach 1

The first approach I took involved adding a column to the `signature_recipient_requests` table for storing the following strings:

- passed
- failed
- has not accessed Docusign

These strings would then be used as the display text in the html.

___

## Approach 2

The second approach I took was to create a separate table for recording the KBA status.

### Steps

1. [Create a new migration](#create-a-new-migration)
    - Run the migration code to create the new table
2. [Create a new Model for interacting with the new table](#create-the-model).
3. [Create a mock Docusign Connect Payload for use in testing](#create-a-new-docusign-connect-payload).
4. [Create recorder helper class](#create-a-recorder-helper-class)

#### Create a new migration

The new migration will create a table for storing information from Docusign. It should include the following columns:

- `id`
- `signature_request_recipient_id` - Foreign key used to identify the signer.
- `id_lookup_result` - result from Docusign.
- `id_lookup_result_created_at` - timestamp from Docusign.
- `id_question_result` - result from Docusign.
- `id_question_result_created_at` - timestamp from Docusign.

**What other types of data can I store here?**

#### Create the new Model

Our model will not necessarily need an `as_json` method - it just needs to do two things: create and read.

#### Create a new Docusign Connect Payload

I created my mock payload by starting a new application in Panda on dev22, and using two of my personal gmail accounts for the email addresses needed in the application. After an application is approved and financing documents have been generated, an email is sent to the provided email addresses which opens a link to Docusign.

Users accessing Docusign are greeted by the KBA page where they are asked to provide their address, phone number, last four of the SSN, and their DOB. If any of this information is incorrect, the user fails authentication. If they pass through this section, then they face a second set of challenge questions. If they fail this then authentication fails.

Docusign sends events of interest (such as login attempts) to Panda each time. If a recipient has failed any of the authentication then a block for ti will be present in the XML. **If a recipient has not failed any of the authentication, then no block will appear.**
**If an envelope has two signers, and one passes all KBA, but the other does not. Then the blocks will appear for both recipients.**

These payloads are kept in the Docusign logs view. To create the mock data (and to test the API pathways in postman) I use the log's xml as a mock file and mock payload for my postman body.

#### Create a Recorder Helper Class

I noticed that many of the existing classes have recorder classes. Take the `SignatureRequestRecipient` class - it consists of a model held in `app/models/`, and a file of helper methods in `lib/docusign/envelope_status/` that deal with the actual recording of the data in the db.

I followed this pattern when creating the `DocusgingKBAStatus` model by writing a  recorder class called `dousign_kba_status_recorder.rb`. This class will contain methods used for saving statuses to the database as well as retrieving them.

**What other methods might I need?**

- I could create variables for extracting the timestamp that comes in the Docusign paylaod for each login status, or I could see if theres a way to extract both the time and the status from the Payload into a hash?

- Given the information from the various Docusign payloads I think that it could be useful to have a method that returns failed, passed, has not accessed Docusign.

To do:

[ ] - Figure out a way to record the desired payload information

[ ] - Write a method to return the total number of times a person has failed login


### Tests for Approach 2

| File | Test | Purpose | Expected Result |
| --- | --- | --- | --- |
| `recipient_status_spec.rb`| Check the result of the Docusign id lookup. | Nokogiri should parse the XML and store the information (pass/fail/nil and the timestamp) we are concerned with as the return of a method. | Depending on the test, we should expect to see `nil`, `Passed`, or `Failed` as the result. |
| `recipient_status_spec.rb`| Check the timestamp associated with an id lookup. | Nokogiri should parse the XML and store the information (pass/fail/nil and the timestamp) we are concerned with as the return of a method. | No matter if a user fails or passes the the id lookup there should be a timestamp associated with the result. |
| `recipient_status_spec.rb`| Check the result of the Docusign questions result. | Nokogiri should parse the XML and store the information (pass/fail/nil and the timestamp) we are concerned with as the return of a method. | If a user passes the id lookup challenge, they are given a second challenge consisting of a series of historical questions tied to each recipient's past. |

Note: these tests are obsolete as I have discovered the power of the attributes at the top of the file. Must make more notes on these. Anyway to mock a DB method we use FactoryBot to create objects that our DB can understand. Essentially you define the paramaters you want, or the various auxillary parameters an object could have associated with it and then call it like so:

```ruby
describe Docusign::EnvelopeStatus::RecipientLoginStatusRecorder do

  subject { described_class.new( failed_login_at_lookup) }

  let(:signature_request) { create :signature_request }
  let!(:signature_request_recipient) {
    create :signature_request_recipient, signature_request: signature_request
  }
  let(:failed_login_at_lookup) { create :recipient_login_status, :with_passed_lookup_status, signature_request_recipient_id: signature_request_recipient.id }

end
```

